## Everything - Week 4
> **Shalom**! 🐧💀 I am genuinely cooked for now 💀🐧I have been dead ahh tired for the past week, and I haven't coded at all. SO let's get started.

### 1. Pararelism vs Concurency
> So I am gonna explain this, which somehow I have never explained despite touching Process and Thread across C and Java 🐧💀

So ever heard about the term "Pararel Universe?" There's a reason it is called Pararel and not Concurent.

Because Pararel means "multiple things at the same time," while Concurent means "**managing** multiple things at the same time."

So let's say you have multiple tasks, you have task A and task B. While you're working on Task A you hit a loading because your internet is slow (REAL 💀🐧) so you open your book and began working on Work B. Then you hit a pause again, you need to make a request to your friend to send their part of the answer. So you "unmount" the task B and go work on task A. 

Is this tiring? Yeah, and that's why in Computer Science (and daily life) we have context of [[Process & Threads|Context Switching]].

This is Concurrency. And there's a reason it is Pararel Universe and not Concurrent Universe 💀🐧

### 2. OS Thread vs Coroutines ("Am I invited 🐧💀? Said Virtual Threads)
> So remember the differences between [[Process & Threads]]? Thread is just an independent flow of execution within a Process.

A Thread is managed by the OS, and it is very expensive (> 1 MB) so you can't stack 50000 OS Thread without `OutOfMemoryError` or... Crash 💀🐧

Kotlin Coroutines led the way earlier than Java, officially 🐧. 

#### Green Threads, Virtual Thread's Brother 🐧🐧

EHEM, Actually 🐧💀

Java Pioneered the N:1 Threading model. Yes, not M:N but M:1, this is why I said "officially."

Java had what we call Green Threads. It basically is Virtual Threads object, but all executed within 1 CPU Core. This was briliant at the time because the OS hasn't natively or matured enough to support Concurrency.

But because we only have 1 Core to use, we have the problem of not utilizing all the available resources, and freezing the one thread 💀🐧So we abandoned it in Java 1.2 and it got removed in Java 1.3

#### Coroutines, Kotlin's answer
> So Kotlin was made with one goal in mind: Make a language that is 100% compatible with JVM while providing better syntax.

So Coroutines was added as Kotlin's answer to scaling up your application to reach concurrency. And boy did they cook.

So basically, similiar to [[Process & Threads - Java|Virtual Threads]], Coroutines is an object of implementation of `ContinuationImpl`. It creates an Object on the Heap.

This Object will be executed in OS Thread Pool.

The difference that is that, your code is transformed by Kotlinc, and it is a pure JVM Code that utlizes JVM Bytecode's features at the time (2018)

#### Virtual Threads, Surviving Brother
> After seeing Kotlin's success, we have Java's answer to fix their OS Thread Problem: [[Process & Threads - Java|Virtual Thread]].

Same as Kotlin's Coroutines, this is an object. But it is managed by the [[JVM Architecture & WORA|Java Virtual Machine]]. This means the one responsible for mounting and unmounting Virtual Threads is not sequential code interuption but the JVM itself.

### 3. Why? - No Idea for title 💀
> Now, the reason why we now mostly use Coroutines instead of Virtual Threads is compatibility.

Virtual Threads allow you to just... write `Executors.newVirtualThreadPerTaskExecutor()` and not worry why would you choose Coroutines? Simple, Compatibility.

Virtual Thread's strength and weakness is the same: It is managed by the JVM.

On one side you need not to worry about anything, just let JVM work.
On the other side it is incompatible when compiling code outside of JVM world.

[[JVM Architecture & WORA|Javac]] only compiles down to JVM Bytecode, while [[Null Safety & Types|Kotlinc]] compiles to JVM, and many more platform. Because Coroutines is tied to Kotlin, and not JVM, Coroutines is the only solution for multi-concurrent application in a fully JVM-Compatible app outside of JVM.... Get it 🐧💀? Basically Multi-Concurrent code that can work with or without JVM.

### 4. How to Coroutines
> So let's take explanation from JavaScript `async`/`await

In JavaScript, `async` function means this function will have pause (`Promise<T>`) and await is where that function got halted.

In Kotlin we have `suspend`, `CoroutineScope` `withContext`, `launch()`, Dispatcher, `await` and `Async`... Only two of those words actually feel familiar huh 💀🐧

- `suspend`: `suspend` keyword only has one job: To mark this function as a Coroutine/Async function.
- `CoroutineScope`: Coroutine Scope is what groups together your Coroutine Objects. So your CoroutineScope will only return if all tasks inside this scope is finished.

The rest need their own sub section.

#### Dispatcher & `withContext`
> Remember the term Thread Pool? We call these thread, Worker Threads.

So Dispatcher is basically traffic cop that reroutes task to Thread Pool.

- `Dispatchers.Main`: Run tasks on the Main UI Thread. Usually we run this on main function
- `Dispatchers.IO`: This is for I/O operation (Surprise 💀🐧) Has the limit of 64 Threads or CPU Core (Which one is larger)
- `Dispatchers.Default`: The number of your CPU's cores. For heavy intensive work
- `Dispatchers.Unconfined`: Unconfined, as its name suggest it doesn't limit the Coroutines to any specific thread. Starts on the current thread and resumes whereever the suspending function finishes.

Now, you see each Dispatcher has their own Threadlimit right? But don't get fooled, they don't create and destroy worker threads on a whim, that is **EXPENSIVE** 🐧💀.

So I don't know what is happening internally, but all of these Dispatchers share the same global pool thread (Probably 64/CPU Cores Amount or which ever is larger.)

Oh yeah, I'll began calling CPU Cores Amount CCA from now on 💀🐧

So now, what is `withContext`? It is the tools you use to switch Dispatcher.

So in Kotlin we use the Main-Safety method. So you declare with `CoroutineScope` on the main function, and in each method you explicitly declare what Dispatcher you need based on CCA.

```kotlin
// Caller doesn't need to know which dispatcher fetchData needs
scope.launch(Dispatchers.Main) {
    showLoadingSpinner()
    
    // fetchData handles its own dispatcher internally using withContext
    val result = fetchData() 
    
    showResult(result) // Still safely on Main thread
}

suspend fun fetchData(): Data = withContext(Dispatchers.IO) {
    // Encapsulated thread-switching
    api.getData()
}
```

#### `launch(), await()` and `async`
> So It is not like Virtual Thread where you leave everything to JVM. You need to fine grained control to Coroutines. This is where launch, await, and Async comes from.

`launch()` basically means launch a NEW Coroutine without any returning value.
`async()` like JavaScript returns a value, but is a waiting operation.

While you need to call await on a value returned from `async`

```kotlin
package io.github.bernardusz.coroutines
import kotlinx.coroutines.*

suspend fun fetchData(): String = withContext(Dispatchers.IO) {
    delay(1000L)
    "Data Returned!"
}

suspend fun loadingSpinner() {
    delay(2000L)
    println("Loading spinner finished")
}

// Uses coroutineScope at the root instead of withContext
suspend fun main(): Unit = coroutineScope {
    // 1. Launch background fetch concurrently
    val data = async { fetchData() }

    // 2. Run loading spinner concurrently so it doesn't block the rest of main
    launch { loadingSpinner() }

    // 3. Print result as soon as data returns
    println(data.await())
}
```
#### `runBlocking()`
> This niche, but needed to be discussed as well. This one defeats the whole purpose of Coroutines: Blocking Thread.

This is used either for testing, blocking the main function until everything inside it is finished (including background world) and integrating old Java syncronous code to Coroutines.

I won't be discussing this 🐧💀 Except that `suspend fun main` equals to `fun main() = runBlocking{}`. And I won't ellaborate. 🐧💀