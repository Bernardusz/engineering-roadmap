---
tags:
  - C
aliases:
  - Pointer
  - Process
  - Threads
  - System
---
# Compilation of my process & thread journey

# Documentation of process & thread - Day 1
> **Shalom**! I feel energized, let's get this started

## 1. Process
> So we already know about virtual memory and virtual address. So what is a process? A process is basically when a program is executed. Like VS Code, for example: When you open the ```.exe``` it spins up a new process with its own new Virtual Memory as we talked in [[Memory|Memory]]. So TLDR: A process = a program running.

## 2. Thread
> So now what is thread? A thread is basically a worker inside each process. It's a sequence of instructions being executed inside processes. So a program doing things is a thread in action.

## 3. CPU Core and Threads
> So now we have heard the words process and thread, but what's the connection to hardware? Simple: core and thread. The number of cores in a CPU basically means how many things a CPU can do simultaneously. So what about the thread when you hear 8 Cores 16 Threads? It means a core has 2 lanes: If one lane is stuck on something slow, like moving data from storage to memory, it switches lanes to do something else. Now, another thing to watch out for is that a 1 Core CPU can feel like it is doing many things at once; this isn't the case, it is called time-slicing. It's just the  CPU switching up between tasks really fast. But this is expensive and slow, because the CPU has to save a snapshot of the current process before moving to the next task. The same as human who takes a lot of energy for context switching, CPU is the same.

## 4. Stack and Heap
> Here we go again, back to memory [[Memory|Memory]]. A process owns its own Virtual Memory (Text, Data, Stack, Heap), which is usually called a sandbox, so when your VS Code crashes, it doesn't affect your Spotify or Clickup since they're isolated. But threads inside a process share the same heap inside the process, but they have their own stack since they have different tasks to do.

## 5. Multi-processing
> So, after we know that a process is a program in action, what does multi-processing mean? It's basically a copy of your program Virtual Memory. So what we need to remember is that each process is isolated, so if one process segfault-ed the other won't get killed by the OS. This is the case for an app where security is the top priority, like Chrome, where if a tab dies, the other won't be affected.
```main.c
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        printf("Child process\n");
    } else {
        printf("Parent process, child PID: %d\n", pid);
    }
    return 0;
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/process$ gcc -o main main.c && ./main
Parent process, child PID: 36861
Child process
```
## 6. Multi-threading
> So now, we're heading to Multi-threading. Multi-threading basically creates a smaller instance of a worker inside a program in order to do multiple things at once. It's faster and cheaper than spinning up a new process because they share the same heap and can communicate easily.
```main.c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h> // For sleep function

// The function that will be executed by the new thread
void *myThreadFun(void *arg) {
    sleep(1); // Simulate some work
    printf("Hello from a new thread!\n");
    return NULL;
}

int main() {
    pthread_t thread_id;
    printf("Before thread creation\n");

    // Create a new thread
    pthread_create(&thread_id, NULL, myThreadFun, NULL);

    // Wait for the created thread to finish
    pthread_join(thread_id, NULL);

    printf("After thread creation (main thread continues)\n");

    return 0;
}
``` 

> However, the danger lies in the shared heap. If two threads try to change the same data, it will trigger a bug known as a data race.

```main.c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
// The function that will be executed by the new thread
void *myThreadFun(void *arg) {
	int *pNum = (int *)arg; // Cast the argument to the appropriate type
	*pNum += 10; // Modify the value pointed to by the argument
	printf("Inside Thread: Value = %d\n", *pNum);
	return NULL;
}

int main() {
	int *pNumber = malloc(sizeof(int));
	*pNumber = 42;
    pthread_t thread_id;
    printf("Before thread creation\n");
    // Create a new thread
    pthread_create(&thread_id, NULL, myThreadFun, pNumber);
    // Wait for the created thread to finish
    pthread_join(thread_id, NULL);
    
	
	printf("After thread creation (main thread continues)\n");
	*pNumber += 5;
	printf("In Main: Value = %d\n", *pNumber);
    return 0;
}
```

> This didn't trigger a data race cause it's explicit what lines run first (sequential), but in production, instead of both happening, this will happen:
```pseudocode
A = 0
In thread 1, A++ -> 1 in local memory and writing it to the heap
In thread 2, A++ -> in local memory and writing it to the heap
A = 1
```

```main.c
#include <stdio.h>
#include <pthread.h>

int counter = 0; // Shared Global Variable (on the Data Segment/Heap)

void* increment_task(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        counter++; // The DATA RACE happens here, since pthread_join only cares about stopping main thread until t1 and t2 finish
    } // Both threads access and modify 'counter' simultaneously without synchronization
    return NULL;
}

int main() {
    pthread_t t1, t2;

    pthread_create(&t1, NULL, increment_task, NULL); /// Create a thread
    pthread_create(&t2, NULL, increment_task, NULL); // Create another thread

    pthread_join(t1, NULL); // Make the main thread wait for t1 to finish
    pthread_join(t2, NULL); // Make the main thread wait for t2 to finish

    printf("Final Counter: %d\n", counter); 
    return 0;
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/process$ gcc -o main main.c && ./main
Final Counter: 1120591
```

> Now, to prevent a data race, we need something called a mutex, which basically lets only one thread modify a value at a time.

```main.c
#include <stdio.h>
#include <pthread.h>

int counter = 0;
pthread_mutex_t lock; // 1. Create the lock, which makes only one thread access at a time

void* safe_increment_task(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&lock);   // 2. Grab the key and if the lock is already taken, wait
        counter++;                   // 3. Increment the counter, one thread at a time
        pthread_mutex_unlock(&lock); // 4. Return the key, and if another thread is waiting, let it proceed
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    
    pthread_mutex_init(&lock, NULL); // 5. Initialize the lock

    pthread_create(&t1, NULL, safe_increment_task, NULL); // Create two threads
    pthread_create(&t2, NULL, safe_increment_task, NULL);

    pthread_join(t1, NULL); // Make the main thread wait for both threads to finish
    pthread_join(t2, NULL);

    pthread_mutex_destroy(&lock); // 6. Clean up the lock for good practice

    printf("Final Safe Counter: %d\n", counter); // Result: Always 2,000,000
    return 0;
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/process$ gcc -o main main.c && ./main
Final Safe Counter: 2000000
```
## 7. Multi-processing vs Multi-threading
> So TLDR: Choose multi-threading if you need performance and speed since they share the same Virtual Memory, but can manage the danger of data race. Choose multi-processing if you need safety and security where one process can't interfere with another. And as a side note, more than 2 processes can interact with each other via Inter-Process Communication; it's far more expensive than multi-threading.

# Documentation of process and thread - Day 2
> **Shalom**! I am lazy, so time to work 💀🐧

## 1. Process Lifecycle & States
> So every process has a lifecycle: ```new``` -> ```ready``` -> ```running``` -> ```waiting``` -> ```terminated``` And there are some ways we can manage it via a function.

### Lifecycle & States

| Process | Explanation |
|----------|--------------|
| New | Happens milliseconds after your program, like ```./main``` is called, the process of spinning out a new process with Virtual Memory, Virtual Address, etc. |
| Ready | Ready is when your program is ready to execute and is put in a queue, waiting for its main entry (```int main()```). Your CPU may still be busy from VS Code and Chrome, so it's here |
| Running | After your program gets its turn, it's executed immediately. |
| Waiting | Now, let's say you need Inter-Process Communication or waiting for a user input that's slow. This is where your program is put on a waiting list, so the CPU can do something else. This is Time Slicing in action, and Context Switching happens a lot in milliseconds.|
| Terminated | When your program reaches ```return 0;``` or ```exit(0)``` your program gets cleaned up, memory is deallocated. |

```main.c
#include <stdio.h>
// NEW Within milliseconds of creation, the process is in the NEW state
// READY After being admitted by the OS scheduler, the process is in the READY state, waiting for its turn to be executed by the CPU
int main() {
    int x = 10;          // RUNNING when the process is executing instructions on the CPU
    char buf[10];
    fgets(buf, 10, stdin); // Transition to WAITING, due to I/O operation which is slow
	x += 5;             // Transition back to READY, waiting for CPU time
    return 0;            // Transition to TERMINATED
}
```

### Ways we can manage Process' Lifecycle & States
> There are some functions available header in which we can get these functions

| Function                                                      | Header       | Explanation                                                                                                                                                                                                                                                                                                                        |
| ------------------------------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ```fork()```                                                  | <unistd.h>   | Fork creates a new process identical to our already existing program process. This is called a child-process.                                                                                                                                                                                                                      |
| ```sleep()```                                                 | -            | It tells the CPU to move the running thread from Running to the Waiting for the specified duration                                                                                                                                                                                                                                 |
| ```wait()```                                                  | <sys/wait.h> | The same as ```sleep()``` but it waits for any child to terminate                                                                                                                                                                                                                                                                  |
| ```waitpid(pid_t pid, int *_Nullable wstatus, int options)``` | <sys/wait.h> | A targeted version of wait, a way for the parent process to wait for a specific child process                                                                                                                                                                                                                                      |
| ```waitid()```                                                | <sys/wait.h> | This is an explicit and modern version of ```waitpid()```. It doesn't just tell you the child died; it can tell you if the child was stopped (paused), continued (resumed), or killed by a specific signal (like a crash).                                                                                                         |
| ```exit()```                                                  | <stdlib.h>   | Now my favorite function, ```exit()```, this basically does a syscall behind the scene to immediately terminate the process. No matter where, it just kills the process. This also flushes all the unused ```printf()``` to the stdio. There's a stronger version ```_exit()``` where it doesn't flush, just Terminate immedately. |
| ```return 0```                                                | -            | This is not a function, but a way to end a function. If this is run on ```int main()``` It will terminate the program. 0 for successful execution, any other number for failure. Effectively calling ```exit(0)```                                                                                                                 |

```main.c
#include<stdio.h>
#include <stdlib.h>
#include <unistd.h> // For fork() and getpid()
#include <sys/types.h> // For pid_t
#include <sys/wait.h> // For wait() and waitpid()

int main() {
	pid_t pid = fork(); // Create a new process
	if (pid > 0) { // Check if we are in the parent process, because in parent the PID of the childprcessis returned
		printf("Hello from parent process\n"); // While it's 0 if it's the child process
	} else { // We are in the child process
		printf("Hello from child process\n");
	}

	// pid < 0 indicates fork() failed
	// pid == 0 indicates we are in the child process
	// pid > 0 indicates we are in the parent process, and the value is the child's PID

	// A process went through the same process: new, ready, running, waiting, terminated
	// Processes run independently and do not share memory space
	// However when they depend on each other, they can use IPC (Inter-Process Communication) mechanisms like pipes, message queues, shared memory, and semaphores
	// This is slow and complex compared to threads
	// Processes are heavier than threads in terms of resource usage and context switching time
	// Processes can wait when they need to access shared resources, waiting for input from human, etc. Leading to potential delays

	// For example, to make the parent wait for the child to finish: we use wait()
	if (pid > 0) {
		wait(NULL); // Parent waits for any child to terminate
		printf("Child process has terminated\n"); // By the time this lin executes, the child has finished
	}

	if (pid > 0) {
		waitpid(pid, NULL, 0); // Wait for a specified child-process to terminate
	}

	if (pid == 0){
		exit(0); // Exit the child process, immediately terminating it. And there's a reason it's in the stdlib, it's special
	}

	if (pid > 0) {
		return 0; // Parent process exits normally
	} 
	else {
		return 0; // Child process exits normally
	}

	return 0; // TERMINATE
}
```

## 2. Copy On Write - ```fork()```
> On man pages, specifically ```man fork```, it says "fork()  creates a new process by duplicating the calling process. " So why when we ```fork()``` the program doesn't immediately take twice as much memory? Simple Copy On Write.

> "The child process and the parent process run in separate memory spaces.  At the time of ```fork()``` both memory spaces have the same content." So they share the same memory at first; the only thing that's new is a new Page Table ([[Memory|Memory]]). The child process has a new Page Table, so both Virtual Addresses in the Parent and Child process points to the same Frame in RAM (Read-only). When a Child Process decides to write, it triggers a Page Fault, the OS intervenes and sees a COW write, and allocates a new Frame in RAM for that specific change.


## 3. Exec family
> So now, how do terminals like Bash and CMD give control to a specific program like Node or Python? The exec family. There are a bunch of exec functions, and all of them do the same thing, but take different parameters: replace the current process image with a new process image. What we'll touch on is ```execlp()```

```main.c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork(); // Create a new process
    if (pid == 0) { // IN CHILD PROCESS
        // execlp(file, command_name, argument, ..., NULL)
        execlp("ls", "ls", "-l", NULL); // Replace child process with 'ls -l'

        // If exec succeeds, THIS LINE IS NEVER REACHED.
        // The child's memory was wiped and replaced by 'ls'.
        printf("This will never print unless exec fails!\n");
    } else {
        // PARENT: Wait for the child (who is now running 'ls') to finish
        wait(NULL);
        printf("Parent: Child is done, I'm still the original program.\n");
    }

    return 0;
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/process$ gcc -o main main.c && ./main
total 20
-rwxr-xr-x 1 bernardus bernardus 16088 Feb 11 18:08 main
-rw-r--r-- 1 bernardus bernardus   687 Feb 11 18:07 main.c
Parent: Child is done, I'm still the original program.
```

## 4. Inter Processes communication
> Now we already know that every process has its own Virtual Memory and is isolated from one another, so how do they communicate? IPC: Inter-Process Communication. In C, we use the ```pipe()``` function. ```pipe()``` basically is just a unidirectional communication pipe, or in other terms, a telephone, with read-only and write capabilities.

```main.c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    int fd[2]; // Create a variable to store the pipe
	// fd[0] is read, fd[1] is write
    pid_t pid; // Create the holder for the process ID
    char buffer[20];

    // 1. Create the pipe BEFORE forking
    if (pipe(fd) == -1) { // Create the pipe and check for errors at the same time
        printf("Pipe failed\n");
        return 1; // Exit if pipe creation fails
    }

    pid = fork(); // 2. Fork the process

    if (pid > 0) { // PARENT
        close(fd[0]); // Close unused read end
        
        printf("Parent: Sending message to child...\n");
        write(fd[1], "Hello Child!", 13);  // Basically a printf you can use to write anywhere, this time we use it to write to the pipe
		// Write is a part of the syscalls, which will be discussed next week
        
        close(fd[1]); // Finished writing
    } 
    else { // CHILD
        close(fd[1]); // Close unused write end
        
        read(fd[0], buffer, sizeof(buffer)); // Read from the pipe
        printf("Child received: %s\n", buffer);
        close(fd[0]); // Finished reading
    }

    return 0;
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/process$ gcc -o main main.c && ./main
Parent: Sending message to child...
Child received: Hello Child!
```

## 5. Thread Synchronization - Mutex & CPU Flushes.
> After many talks about the process. Time to do the dangerous and fast sibling: thread. So Threads share everything but the Stack. So how do parent and child synchronize the same Data/Heap segment?

> In the previous comment, we talked about Mutex, which is exactly the way to prevent data race and wait for the CPU to synchronize both data. 

```main.c
#include <stdio.h>
#include <pthread.h>

int counter = 0;
pthread_mutex_t lock; // 1. Create the lock, which makes only one thread access at a time

void* safe_increment_task(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&lock);   // 2. Grab the key and if the lock is already taken, wait
        counter++;                   // 3. Increment the counter, one thread at a time
        pthread_mutex_unlock(&lock); // 4. Return the key, and if another thread is waiting, let it proceed

		// Now every time the mutex is unlocked, the CPU flushes the cache to the RAM (Frame), so the next thread that locks it will see the updated value of the counter.
		// This ensures that both threads will correctly increment the counter
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    
    pthread_mutex_init(&lock, NULL); // 5. Initialize the lock

    pthread_create(&t1, NULL, safe_increment_task, NULL); // Create two threads
    pthread_create(&t2, NULL, safe_increment_task, NULL);

    pthread_join(t1, NULL); // Make the main thread wait for both threads to finish
    pthread_join(t2, NULL);

    pthread_mutex_destroy(&lock); // 6. Clean up the lock for good practice

    printf("Final Safe Counter: %d\n", counter); // Result: Always 2,000,000
    return 0;
}
```

## 6. Scheduling intuition
> So now let's get real low-level 🐧

### Cores and Threads.
> So How many cores in your CPU decides how many things can happen at the same time (True parallelism), while the thread in your CPU spec is just for Hyper Threading, so if in Core 1, Thread A is slow, it can switch to Thread B real quick.

> The OS actually lies to the CPU. The CPU doesn't understand what a process is; it only understands math and logic. For the CPU, it's a never-ending sequence of tasks.

### Scheduling, Time Slicing, and Context Switching
> So if you have a limit on how many things you can do simultaneously, how does the OS handle this? As I said, scheduling. The OS switches between Thread A, B, and C very fast, within milliseconds. This is Time Slicing, and results in Context Switching. The CPU must save a bunch of things I'm not gonna deep dive into 🐧💀. As an analogy, if a human context switches, they need to save their current work, and remember it or write it somewhere else before doing another task. It's expensive, it's heavy, and it's slow. That's why too many threads don't result in a performance gain; instead, it's a loss.

> However, modern schedulers inside the OS, which tell the CPU which Thread to execute, must decide which Thread goes next. Linux uses the Completely Fair Scheduler (CFS); every Thread got a Fair share of CPU time, not equal, but fair based on needs. Another analogy, if you have already eaten in a retreat, while your friend hasn't, the food will be given to you later if your friend has eaten. In other terms, feed those who have eaten the least.

> But there is also Preemption, analogy again. If your time in the internet cafe is up, you're kicked out. The same as thread time, if a thread's time has run out, the CPU time-slice is given to another thread.

# Mini Parallel File Processor CLI

> Link to the repository: https://github.com/Bernardusz/Mini-Parallel-File-Processor-CLI

> A character and line counter for .txt file, written in C. Taking advantage of Multi-processing and Multi-threading for a fast performance. A part of my Process and Thread material in C - 2.5 Year Plan

## The Library Used

* <stdio.h>: For standard I/O operation to the stdout & FILE, fopen, fclose and fseek
* <stdlib.h>: For exit, etc.
* <sys/types.h>: For types of pid_t
* <sys/wait.h>: For the wait family
* <sys/stat.h>:  for stat, to check the size of files in byte
* <unistd.h>: For fork, and process stuffs
* <pthread.h>: For threading

## How it works (Pseudo)

1. Check if argc > 1
2. Create a pair of pipes for each child process (0 for read, 1 for write)
3. Create a child process for each file
4. In each child process, check whether the file is big enough to justify multi-threading
5. If it is big enough, create a thread, mutex, and divide workload.
6. In each thread, use local stack, before locking the mutex and changing the global value
7. After all threads have finished or the only thread has finished, send the result using pipe.
8. The parents read the data from the pipe, and then waitpid to make sure the child has been terminated before moving to the next child. Repeat.
9. The parent aggregates the result, adding each child's result is added to the final result.
10. BOOM 🐧💀

## Reality Check

> I used AI, I can't write this code from scratch once more. However, I understand what each and every line does, I will prove it in my video later. But my final result is not to be a Kernel dev, just a better programmer who understand how machine works, so my Spring Boot app can function optimized and robustly 🐧🐧🐧🐧.

Chore: Create a video.