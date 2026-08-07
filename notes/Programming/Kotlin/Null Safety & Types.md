
## No More Boilerplate - Day 1
> **Shalom!** 💀🐧 Welcome back to the learning by material and not by project 🐧🐧🐧🐧

### 1. No More `public static void String main()` boilerplate 🐧🐧
> So to start things off, we need to know what the JVM is. Java Virtual Machine is the one interpreting [[JVM Architecture & WORA | bytecode]], while [[JVM Architecture & WORA | javac]] is the one "compiling" Java files into .class (bytecode). In Kotlin, we have kotlinc.

So, because JVM doesn't care about anything other than receiving and running Bytecode (`.class`), you can use multiple JVM languages like Java, Scala, Groovy, and Kotlin. 

And a bit off topic, most programming languages, especially AOT languages, have their way of being converted from readable code to instructions for the CPU in 3 parts: Frontend, Middleend, and Backend, all having different purposes.

For example, GCC has different frontends (`gcc` for C, g++ for C++), and the middle end is for optimizing the code structure, while the backend is for transforming code into machine code.

Clang is also like that, with clearer separation of concerns. Clang is the frontend whose job is to turn `.c` files into LLVM-IR. This is fed into the Middle end (LLVM-Optimizer), which is then fed into the backend that turns LLVM into Machine code.

Java with `javac` and Kotlin with `kotlinc` are the same. They're the frontend for their respective languages, that produces .class bytecode. When fed into the JVM, the JVM will immediately execute the Bytecode. Frequently executed code/method called [[JVM Architecture & WORA | Hotspot]]. When fed into the JIT, the Bytecode passes the Middle end to turn the Bytecode into IR, before being fed to the backend to transform into Machine code.

Again, this is my own understanding. Correct me if I am wrong. I am learning in public, not teaching 🐧🐧🐧🐧

Got some of my information from here: <https://www.scaler.com/topics/java-source-code-is-compiled-into/>

---

SO precisely because of this clear separation between Frontend compiler and JVM, Kotlin can remove many weights of the Java language like main entry filled with boilerplate, while simultaneously adding multiple features like Coroutines and Null safety while retaining 100% interoptability with Java and other JVM languages.

Long live `fun main()` 🐧🐧🐧🐧.

### 2. There is only one type, because of the smart Compiler 🐧🐧🐧
> In java we have [[Collections, Advanced Types & The Memory Cost | primitives]] and [[Collections, Advanced Types & The Memory Cost | Object wrapper]]. This adds the capability that Integer can be `null`, not that it is very good. But still every `List` and `Map` stores the Object wrapper, not the primitives.

In Kotlin, we have type inference. Even though Java has `var`  now, in Kotlin, we fully rely on type inference; `val` for immutable **value**, `var` for mutable **variable**.

But that is not why we're here today. In Kotlin, unlike Java, we do not have `int` and `Integer,` we only have `Int` that the compiler decides it primitive ot object.

There are a lot more types, but mostly just look it up on <https://kotlinlang.org/docs/kotlin-tour-basic-types.html>

### 3. Collections... reduced? 🐧
> So based on <https://kotlinlang.org/docs/kotlin-tour-collections.html> we have 3 kinds of Collections in Kotlin. You can still import Java's Collections or somehow use Kotlin's `HashMap`, but on the documentation site, these are the three available ones;


| Collection | Description                                                        | Immutable version      | Mutable version                      |
| ---------- | ------------------------------------------------------------------ | ---------------------- | ------------------------------------ |
| Lists      | Ordered Collcetion of Item that allows duplicate                   | `List` with `listOf()` | `MutableList` with `mutableListOf()` |
| Set        | Store unordered items and doesn't allow duplicate                  | `Set` with `setOf()`   | `MutableSet` with `mutableSetOf()`   |
| Map        | Store items as key-value pairs that don't allow duplicates for key | `Map` with `mapOf()`   | `MutableMap` with `mutableMapOf()`   |

All of these have their corresponding Java collection inside. And this is done so we don't need to convert Kotlin's special Collection into a Java type; The underlying collection is the same.

### 4. Lambda - Function overcharged
> I know Lambda is great, but sometimes it's too confusing 💀💀, I'll try explaining it slowly, but here we go.

You can store a lambda in a variable. So, like JavaScript, we can store a whole function inside a variable. This is because function itself is also [a type](https://kotlinlang.org/docs/kotlin-tour-functions.html#function-types).

Similar to JavaScript, the type for a lambda/function that takes a String and returns a String is `(String) -> String`. SO technically, a function can return a lambda. Not sure why you'd use that, since I am still in my first hour of Kotlin, but we'll see later.

You can also call a lambda immediately after declaring, just by adding `()`.

And trailing lambda means, if the only parameter is the lambda, you can omit the empty parentheses and immediately follow the function with `{}`. This is also interoperable with Java Consumer, so calling a Java method that accepts a consumer can also be followed with a trailing lambda.

## 5. Class - Open and Simultaneously not? 🐧?
> So now, I am a bit shocked when learning about OOP in Kotlin, but now it makes sense.

#### Classes are Public by default and Final by default
> Most of the time, are we building a package private or public method/class? We create a public so that users can access it. We rarely use package private in Java except for internal testing. So that's why Kotlin's classes and methods are public by default.

At the same time, class is final by default, so nobody can extend/inherit your class unless explicitly allowed via open.

#### Removed Package Private?
> Normally, in Java, and what I use daily for building [Levtus](https://github.com/Bernardusz/levtus), because I need to be able to test private internal methods without exposing them to Developers. But anyone can just create the same structure and use it. So that's why the `internal` keyword exists.

It basically means: "Anyone with the same module (Maven project/Gradle project/IntelliJ IDEA Module) can access this. Not an outsider." Which is far more useful and secure than package private.

#### Record-like way to define properties
> In Kotlin, you have two ways of declaring properties:

```kotlin
class Contact(val id: Int, var email: String = "example@gmail.com") {
    val category: String = "work"
}
```

The first one is inside the parentheses after the class name, or inside the class body.

### 6. Null Safety - No No Penguin is allowed 🐧🐧🐧🐧
> Now, when I look at Kotlin Null Safety, I am grateful I have learned JavaScript & TypeScript cause it is very similar; `String` for non-nullable, and `String?` for nullable. And Non-nullable doesn't accept nullable.

Just like TypeScript, when calling a method on a nullable, to be safe, use the `?.` to be safe, and it will return `null` if the argument is also `null`.

But in Kotlin, we have what is called the Elvis operator. If the left side is null, what is on the right of the Elvis operator will be returned instead.

That is it for day 1 🐧🐧🐧🐧
