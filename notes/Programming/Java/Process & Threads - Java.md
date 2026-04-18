## Threads - Day 1
> **Shalom!** 🐧, I am tired. I slept only 4-5 hours last night 🐧💀 But the learning shall goes on. SO let's finish this ASAP

### 1. "Where is my brother, 'Process'!? - Thread, Probably 🐧💀
> So yeah,in Java we can technically do what [[Process & Threads|fork() - Process]] do in Java, but in Java, JVM is just a normal [[System Calls (Syscalls)|Ring 3]] process, and inside the JVM your code is just being run by the JVM, with a bunc of workers (threads) inside.
> If Iḿ not mistaken, I said Java is a nested process, process inside a process (JVM), it's wrong, ypur code is interpreted inside the JVM as one or more Threads. I didn't know where, But I was wrong 🐧🙏

### 2. Thread & Runnable - C's Thread 🐧
> So in C we pass a pointer function correct? Which means we pass the *task* to the thread. In Java the logic is the same.

Runnable is basically that pointer function, you create a algorithm or set of tasks inside it, and then you will pass it to the `Thread`, because remember, Java is OOP not FP.

```java
public class Main {
    public static void main(String[] args) {
        // 1. Define the TASK (Like your function pointer in C)
        Runnable task = () -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("Hello from " + threadName);
        };

        // 2. Define the WORKER (The pthread_t equivalent)
        Thread worker = new Thread(task);

        // 3. Start it (This triggers the OS-level thread creation)
        worker.start();
    }
}
```

### 3. Syncronization - Data Race 🏎🐧
> So remember [[Process & Threads|Mutex]] ? In Java Mutex is located in the [[4 Pillars of OOP & Memory Truth|Mark Word]] of every Object. But merely creating a `pthread_mutex_t lock;` doesn't stop data race. That's where `syncronized` keyword comes in.
> `syncronized` makes sure whatever is done here will pause the execution of the other Threads.

```java
public class Main {
    private static RandomInteger counter = new RandomInteger(0);
    public static void main(String[] args) {
        // 1. Define the TASK (Like your function pointer in C)
        Runnable task = () -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("Hello from " + threadName);
            int localCounter = 0;
            for (int i = 0; i < 1000000; i++) {
                localCounter++;
            }
            counter.updateData(localCounter);
            System.out.println("Counter value: " + counter.getValue());
            System.out.println("Finished thread " + threadName);
        };

        // 2. Define the WORKER (The pthread_t equivalent)
        Thread worker = new Thread(task);
        Thread secondWorker = new Thread(task);
        Thread thirdWorker = new Thread(task);
        // 3. Start it (This triggers the OS-level thread creation)
        worker.start();
        secondWorker.start();
        thirdWorker.start();
    }
}

class RandomInteger{
    private int value;
    public RandomInteger(int value) {
        this.value = value;
    }
    public void updateData(int data) {
        synchronized(this) {
            this.value += data;
        }
    }
    public int getValue() {
        return this.value;
    }
}
```

```bash
Hello from Thread-1
Hello from Thread-2
Hello from Thread-0
Counter value: 1000000
Finished thread Thread-0
Counter value: 2000000
Finished thread Thread-1
Counter value: 3000000
Finished thread Thread-2
```

However as you can see, the sycronized is passed a refrence to the object. When a Thread starts executing the method, it doesn't stop other threads yet. Instead it waits for which Thread hits the line of `syncronized(this)` first.

If you want to immediately suspend every Thread other than the first Thread to hit this method, just put the syncronized in the method name.

```java
public class Main {
    private static RandomInteger counter = new RandomInteger(0);
    public static void main(String[] args) {
        // 1. Define the TASK (Like your function pointer in C)
        Runnable task = () -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("Hello from " + threadName);
            int localCounter = 0;
            for (int i = 0; i < 1000000; i++) {
                localCounter++;
            }
            counter.updateData(localCounter);
            System.out.println("Counter value: " + counter.getValue());
            System.out.println("Finished thread " + threadName);
        };

        // 2. Define the WORKER (The pthread_t equivalent)
        Thread worker = new Thread(task);
        Thread secondWorker = new Thread(task);
        Thread thirdWorker = new Thread(task);
        // 3. Start it (This triggers the OS-level thread creation)
        worker.start();
        secondWorker.start();
        thirdWorker.start();
    }
}

class RandomInteger{
    private int value;
    public RandomInteger(int value) {
        this.value = value;
    }
    public synchronized void updateData(int data) {
        this.value += data;
    }
    public int getValue() {
        return this.value;
    }
}
```

```bash
Hello from Thread-0
Hello from Thread-2
Hello from Thread-1
Counter value: 1000000
Finished thread Thread-1
Counter value: 2000000
Finished thread Thread-2
Counter value: 3000000
Finished thread Thread-0
```

### 4. Reentrancy - "I will not lock myself" - 🐧🔒
> So let's get back to C time. In C, Mutex is a simple lock or unlocked. When Thread A locks the Mutex, and somehow inside that same function it tries to lock the Mutex, it will hang forever.
> Because Thread A is waiting for the instruction to finish, but the instruction is also waiting because it is used by Thread A 🐧💀
> In Java, you don´t just store locked or not. You have the owner and counter. If inside the same Object you call syncronized methods, instead of locking it will increment the counter to 2 syncronized with the owner of this method.
> So no Hanging until the end of time, work 🐧💯

```java
public class Main {
    public static void main(String[] args) {
        // 1. Define the TASK (Like your function pointer in C)
        Runnable task = () -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("Hello from " + threadName);

            outer();

            System.out.println("Finished thread " + threadName);
        };

        // 2. Define the WORKER (The pthread_t equivalent)
        Thread worker = new Thread(task);
        Thread secondWorker = new Thread(task);
        Thread thirdWorker = new Thread(task);
        // 3. Start it (This triggers the OS-level thread creation)
        worker.start();
        secondWorker.start();
        thirdWorker.start();
        
        try{
            worker.join();
            secondWorker.join();
            thirdWorker.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public static synchronized void outer() {
        System.out.println("Entered outer");
        inner(); // This calls another synchronized method on 'this'
        System.out.println("Exiting outer");
    }

    public static synchronized void inner() {
        System.out.println("Inside inner");
    }
}
```

```bash
Entered outer
Inside inner
Exiting outer
Finished thread Thread-1
Entered outer
Inside inner
Exiting outer
Finished thread Thread-2
Finished thread Thread-0
```

### 5. Thread Lifetime 🐧⏲
> In [[Process & Threads]] we know that in OS, thread or an execution has 5 phases: `new`, `ready`, `running`, `waiting`, and `terminated`. In Java it's different.

| State         | C Equivalent / Analogy | What is happening?                                                                                      |     |
| ------------- | ---------------------- | ------------------------------------------------------------------------------------------------------- | --- |
| NEW           | Before pthread_create  | The Thread object exists in the heap, but start() hasn't been called yet. No OS thread exists.          |     |
| RUNNABLE      | READY + RUNNING        | The thread is executing in the JVM, or it is ready and waiting for the OS to give it a CPU time-slice.  |     |
| BLOCKED       | Waiting for a Mutex    | The thread is waiting to enter a synchronized block because another thread holds the lock.              |     |
| WAITING       | wait() or join()       | The thread is waiting indefinitely for another thread to perform a specific action (like pthread_join). |     |
| TIMED_WAITING | sleep(ms)              | The thread is waiting for a specified period (e.g., Thread.sleep(1000)).                                |     |
| TERMINATED    | exit() / Finished      | The thread has completed its run() method or was stopped due to an exception.                           |     |

> No Running? Running and Ready are merged into RUNNABLE. Java is [[JVM Architecture & WORA|modular]] remember? It leaves the interpretation of Running and Ready to the User's [[System Calls (Syscalls)|Operating System]].

```java
public class Main {
    public static void main(String[] args) {
        // 1. Define the TASK (Like your function pointer in C)
        Runnable task = () -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("Hello from " + threadName);
            System.out.println("Hello I am " + Thread.currentThread().getState());

            System.out.println("Finished thread " + threadName);
        };

        // 2. Define the WORKER (The pthread_t equivalent)
        Thread worker = new Thread(task);
        Thread secondWorker = new Thread(task);
        Thread thirdWorker = new Thread(task);
        // 3. Start it (This triggers the OS-level thread creation)
        worker.start();
        secondWorker.start();
        thirdWorker.start();

        try{
            worker.join();
            secondWorker.join();
            thirdWorker.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```bash
Hello from Thread-1
Hello from Thread-0
Hello from Thread-2
Hello I am RUNNABLE
Hello I am RUNNABLE
Hello I am RUNNABLE
Finished thread Thread-2
Finished thread Thread-1
Finished thread Thread-0
```

RUNNABLE because `getState()` is called when... You guessed it, the Thread running.

### 6. Interruption - No Guillotine 🐧🔪
> In C you can essentially purge Thread in the middle of their run. But as you can guess: Unsafe 🐧💀
> So in Java `thread.stop()` was deprecated and now we use `thread.interrupt()`.
> But don´t get your hopes up, It's just a toggle for Boolean that the Thread will read. And if it is set to true, then it would throw `InterruptedException`, which must be handled using `catch`

```java
public class Main {
    public static void main(String[] args) {
        // 1. Define the TASK (Like your function pointer in C)
        Thread worker = new Thread(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                try {
                    // Do work
                    System.out.println("Working hard...");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    // IMPORTANT: sleep() throws this if interrupted!
                    System.out.println("I was interrupted while sleeping! Cleaning up...");
                    // Standard practice: preserve the interrupt status
                    Thread.currentThread().interrupt();
                    break; // Exit the loop
                }
            }
            System.out.println("Worker thread safely shut down.");
        });

        try{
            worker.start();
            Thread.sleep(5000);
            worker.interrupt();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```bash
Working hard...
Working hard...
Working hard...
Working hard...
Working hard...
I was interrupted while sleeping! Cleaning up...
Worker thread safely shut down.
```

It's 11:07 PM 🐧💀 I need to sleep, So I am sorry if the grammar is very, very bad 🐧🙏

## Easy Way for Threads 🐧🛹 - Day 2
> **Shalom**! I'm still on 5 hours of sleep, I got a short story to write for assignments, another assignment... and yeah 🐧💀 So, let's finish this ASAP.
> 
### 1. ExecutorService - The Benevolent Manager 🐧📑
> So Thread is a worker, and in C and Java, you need to be the CEO, Manager, and HRD... Tiring and prone to errors 🐧💀
> So Java has an Object called ExecutorService that acts as a Manager. You drop the tasks it delegates tasks between workers.
> So instead of a Thread/Worker doing a task and then get killed 🐧💀 That Thread will instead work on other tasks.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class Main {
    public static void main(String[] args) {
        // 1. Define the TASK (Like your function pointer in C)
        ExecutorService executor = Executors.newFixedThreadPool(4);

        for (int i = 0; i < 10; i++) {
            executor.submit(() -> {
                System.out.println("Task is running");
            });
        }

        executor.shutdown();

        try {
            if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}
```

Why the try? Because it means, wake me (The Main Thread) when a Thread called `mainThread.interrupt()`. Which as we learned, how do you wake up a Thread immediatelt? Yup, Bonk Him with an `InterruptedException` 🐧🔨

So other than executor signaling all Threads are closed or 1 minute has passed, a Thread inside worker can call interrupt on the mainThread, so that's why the try-catch

### 2. Atomic Variable - CPU Life Hack 🐧🧠
> I drank chocolate milk, now I am sleepy, too sleepy for my own good 🐧💀, Let's go.
> So in [[Process & Threads|Process & Threads]] we learned about Context Switching between Threads is expensive. The same with Java's Thread.
> So we have what's called Atomic Variable. Instead of switching Thread's state to BLOCKED, we do a special CPU's instruction: Compare-And-Swap (CAS). It's basically optimistic approach, it treis to switch first and handle error later.
> TLDR; "Go to this variable and increment the value by 1 if the value is still 10, if it's changed, re do it until succesful."

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

public  class Main {
    public static void main(String[] args) {
        Stats stats = new Stats();

        ExecutorService executor = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++){
            executor.submit(() -> {
                stats.handleRequest();
            });
        }
        executor.shutdown();
    }
}

class Stats {
    // This lives in the Shared Heap
    private final AtomicInteger requestCount = new AtomicInteger(0);

    public void handleRequest() {
        // No 'synchronized' keyword needed!
        // This uses the CPU's CAS instruction under the hood.
        int currentTotal = requestCount.incrementAndGet();

        System.out.println("Handling request #" + currentTotal);
    }
    public int getValue() {
        return requestCount.get();
    }
}
```

But if you have 99 threads, only one Thread wins and the other 99 competes again. So don't use this for everything, only uses it when the task is small and the cost of retry is far smaller than nanoseconds of CPU's blocking.

But remember that every non-primitive variable in Java is a [[JVM Architecture & WORA|Reference]], and a String is immutable, and it just changes the Reference to the [[Collections, Advanced Types & The Memory Cost|String Pool]]? That's why there's ` AtomicReference` , so you can decide, which one. 

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicReference;

public  class Main {
    public static void main(String[] args) {
        Stats stats = new Stats();

        ExecutorService executor = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++){
            executor.submit(() -> {
                stats.handleRequest();
                stats.handleResponse();
            });
        }
        executor.shutdown();
    }
}

class Stats {
    // This lives in the Shared Heap
    private final AtomicInteger requestCount = new AtomicInteger(0);
    private final AtomicReference<String> response = new AtomicReference("Initial Value");
    public void handleRequest() {
        // No 'synchronized' keyword needed!
        // This uses the CPU's CAS instruction under the hood.
        int currentTotal = requestCount.incrementAndGet();

        System.out.println("Handling request #" + currentTotal);
    }
    public void handleResponse() {
        int currentTotal = requestCount.incrementAndGet();
        response.set("Response #" + currentTotal);
        System.out.println(response);
    }
    public int getValue() {
        return requestCount.get();
    }
}
```

### 3. Concurrent Collection - ConcurentHashMap🐧🖥 🐧🖥 🐧🖥
> So remember [[Collections, Advanced Types & The Memory Cost|Collections]]? In Java they are not Thread-safe. You can bootstrap your way if you want to use ArrayList in a multi-threaded environtment. But for most of the jobs can be tackled with ConcurentHashMap.

This is very complicated, but think of ConcurentHashMap as [[Memory|Page Table]]. So let's say a HashMap of Books is accessed by 4 Threads at the same time. If Thread-A access Book-1 which is located on Bucket 1 (A page), only that bucket is locked. 

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public  class Main {
    public static void main(String[] args) {
        ConcurrentHashMap<String, String> map = new ConcurrentHashMap<>();
        Runnable runnable = () -> {
          String threadName = Thread.currentThread().getName();
          map.putIfAbsent(threadName, "Hello from " + threadName);
          System.out.println(map.get(threadName));
          System.out.println(map);
          System.out.println("-------------------------");
        };
        ExecutorService executor = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++){
            executor.submit(runnable);
        }
        executor.shutdown();
    }
}
``` 

Why `putIfAbsent`? put is a normal operation that could lead to zombie overriding between threads. Whereas putIfAbsent uses the Compare-And-Swap, which will make sure you are save.

### 4. CountDownLatch - "How many chances left?" 🐧❓
> So, let's say you have a use case that can't use Executor, and you need a way to manage workers efficiently instead of blind `.join`? This is where `CountDownLatch` comes in.
> An atomic counter that make sure your Main-Thread can run immediately after all the prepping Thread is finished.

```java
import java.util.concurrent.CountDownLatch;

public class Main {
    public static void main(String[] args) throws InterruptedException {
        // 1. Initialize with the number of tasks to wait for
        CountDownLatch latch = new CountDownLatch(2);

        // 2. Start Worker 1 (Database)
        new Thread(() -> {
            System.out.println("Connecting to Database...");
            // Simulate work
            latch.countDown(); // The "Buzzer" (-1)
        }).start();

        // 3. Start Worker 2 (Config)
        new Thread(() -> {
            System.out.println("Loading Config files...");
            // Simulate work
            latch.countDown(); // The "Buzzer" (-1)
        }).start();

        // 4. THE WAIT
        System.out.println("Main thread waiting for setup...");
        latch.await(); // This BLOCKS until counter hits 0

        System.out.println("All systems GO! Starting your app.");
    }
}
```

## Modern Way of Threads 🧠🐧- Day 3
> Yesterday we talked about Easy way, today we're doing Easy and Modern way of Threads. And yes I have slept, though it's not enough 🐧💀

### 1. Future & Callable - "Can I order a <Burger>?" 🍔🐧
> Do you understand Async in Java? Yep this is similiar to that but for Threads. You declare what you want from a Thread and it will give it to you when it's ready, otherwise, wait ⏰🐧

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        Runnable task = () -> {
            System.out.println("Task is running");
        };

        Callable<String> callable = () -> {
            Thread.sleep(2000); // Simulating heavy DB query
            return "User Profile Data: Admin";
        };

        Future<String> receipt = executor.submit(() -> {
            Thread.sleep(2000); // Simulating heavy DB query
            return "User Profile Data: Admin";
        });

        Future<String> secondReceipt = executor.submit(callable);



        System.out.println("Doing other things while DB is queried...");

        String result = receipt.get();
        String secondResult = secondReceipt.get();

        System.out.println("Got the data: " + result);
        System.out.println("Got the data: " + secondResult);
        executor.shutdown();
    }
}
```

### 2. Wait/Notify & Condition - "Notify me when you need my help" 🐧🥊
> Why the boxing gloves? I don't know myself 🐧💀, But remember that CPU is a hyperactive child as we talked about in [[System Calls (Syscalls)|System Calls]] 💀? Yep, so it wants to use every nanosecond it has to do something.
> So this is where Wait/Notify comes in:

#### Wait/Notify - The Old Way
> So before condition was introduced, this is how do we say "let me sleep, and when you have task for me, call me" 🛏🐧

```java
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(2);
        SharedBuffer buffer = new SharedBuffer();
        executor.submit(() -> {
            try {
                buffer.produce();
                buffer.produce();
                buffer.produce();
                buffer.produce();
                buffer.produce();
                buffer.consume();
                buffer.consume();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        executor.shutdown();
    }
}

class SharedBuffer {
    private final int LIMIT = 5;
    private int count = 0;

    // You MUST be inside a synchronized block to call wait/notify
    public synchronized void produce() throws InterruptedException {
        while (count == LIMIT) {
            wait(); // "I'm dropping the lock and napping until space opens"
        }
        count++;
        System.out.println("Produced: " + count);
        notifyAll(); // "Hey everyone, I just added data, check if you can work!"
    }

    public synchronized void consume() throws InterruptedException {
        while (count == 0) {
            wait(); // "I'm dropping the lock and napping until data arrives"
        }
        count--;
        System.out.println("Consumed: " + count);
        notifyAll(); // "Hey everyone, I just freed space!"
    }
}
```

#### Condition - The New Way
> The problem with Wait/Notify is that you only have 2 methods for waking other Threads: `notify()` which notifies random people, or `notifyAll()` which, will literally wake everyone up
> Condition allows you to create rooms, for example: In a house there would be someone wanting to use the bathroom. When the bathroom is free you only notify those who needs to use it, not the one who wants to use the kitchen 🐧💀

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newFixedThreadPool(2);
        Room room = new Room();
        executor.submit(() -> {
            try {
                room.useBathroom();
                room.useKitchen();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        executor.shutdown();
    }
}

class Room {
    private final Lock lock = new ReentrantLock();

    private boolean isBathroomOccupied = false;
    private boolean isKitchenOccupied = false;

    private final Condition bathroomAvailable = lock.newCondition();
    private final Condition kitchenAvailable  = lock.newCondition();

    public void useBathroom() throws InterruptedException {
        lock.lock();
        try {
            while (isBathroomOccupied) {
                bathroomAvailable.await();
            }
            isBathroomOccupied = true;
            System.out.println(Thread.currentThread().getName() + " is using the bathroom.");

            Thread.sleep(500);

            isBathroomOccupied = false;
            bathroomAvailable.signalAll();
        }
        finally {
            lock.unlock();
        }
    }

    public void useKitchen() throws InterruptedException {
        lock.lock();
        try {
            while (isKitchenOccupied) {
                kitchenAvailable.await();
            }
            isKitchenOccupied = true;
            System.out.println(Thread.currentThread().getName() + " is using the kitchen.");

            Thread.sleep(500);

            isKitchenOccupied = false;
            kitchenAvailable.signalAll();
        }
        finally {
            lock.unlock();
        }
    }
}
```

### 3. Virtual Threads - Infinite Penguin 🐧🐧🐧🐧
> Everything we learned exists because Threads are expensive, like real expensive. Not as expensive as a process, but expensive still.
> But now, in Java 21, we have what's called Virtual Threads.
> As an analogy, do servers at restaurant wait for you to finish eating? Hell naw 🐧💀 They dissapear and serve other tables.
> So basically Virtual Thread is a small Java object mounted to a few OS Threads.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    public static void main(String[] args) throws Exception {
        ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
        Room room = new Room();
        for (int i = 0; i< 100000; i++){
            executor.submit(() -> {
                try {
                    room.useBathroom();
                    room.useKitchen();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }

        executor.shutdown();
    }
}

class Room {
    private final Lock lock = new ReentrantLock();

    private boolean isBathroomOccupied = false;
    private boolean isKitchenOccupied = false;

    private final Condition bathroomAvailable = lock.newCondition();
    private final Condition kitchenAvailable  = lock.newCondition();

    public void useBathroom() throws InterruptedException {
        lock.lock();
        try {
            while (isBathroomOccupied) {
                bathroomAvailable.await();
            }
            isBathroomOccupied = true;
            System.out.println(Thread.currentThread().getName() + " is using the bathroom.");

            Thread.sleep(500);

            isBathroomOccupied = false;
            bathroomAvailable.signalAll();
        }
        finally {
            lock.unlock();
        }
    }

    public void useKitchen() throws InterruptedException {
        lock.lock();
        try {
            while (isKitchenOccupied) {
                kitchenAvailable.await();
            }
            isKitchenOccupied = true;
            System.out.println(Thread.currentThread().getName() + " is using the kitchen.");

            Thread.sleep(500);

            isKitchenOccupied = false;
            kitchenAvailable.signalAll();
        }
        finally {
            lock.unlock();
        }
    }
}
```