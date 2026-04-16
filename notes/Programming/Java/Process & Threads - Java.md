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