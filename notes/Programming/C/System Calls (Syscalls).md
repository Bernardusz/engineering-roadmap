---
aliases:
  - System
  - OS
  - Low level
tags:
  - C
---

# Compilation of my syscalls journey

# Documentation of System Calls in C - Day 1:
> We have learned about Virtual Memory, Process, and Thread. Now here comes the fun part: Syscalls 🐧💀

## 1. Operating System Spaces
> In OS, there are 2 spaces in which they do different things: Kernel & User Space. The distinction was there so that when a program goes rogue, like dereferencing memory that's not yours, or UB [[Undefined Behavior|Undefined Behavior]], The Kernel can catch it and immediately kill your program.

## User Space
> This is a safe space, the place where all processes live, your VS Code, Chrome, and Clickup all live here, inside their own Virtual Memory.

## Kernel Space
> Kernel Space is a space where it interacts with your hardware directly. Let's say it's a part of your OS, which talks to the CPU. In here lives the scheduler, etc. When your program needs to do something relating to hardware, it needs to go through here.

## Hardware
> **Not a space** But it's a part of your computer in which the Kernel interacts in favor of the program. In Linux distros, Linux is not the OS; it's a Kernel which talks to the hardware. On top of that is GNU, which is the User Space used for processes and users to interact with the hardware. That's why GNU Compilers are still tightly coupled with Linux; it's due to their close nature and historical effect.

## 2. System Calls - The Kernel "API"
> So if Processes are trapped in User Space with no power, how do they do something like save a file? System Calls. System Call is like an API, in which you can call in your program, then the Kernel will guarantee safety and do your command. There are layers in which your request goes down.

| Number | Layer                       | Explanation                                                                                                                                                                                                                                              |     |
| ------ | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| 5      | Interface                   | This is the high-level API, in which you can call in your C program, a famous example is ```prinf()```. And for the sake of learning, let's use ```printf()``` as an example                                                                             |     |
| 4      | I/O Buffer/Device Drivers   | This is the next layer, in which you call to do I/O operations, still a bit high-level, ```printf()``` just calls ```write()```, and uses it to print to the stdout. And it interacts with I/O devices like a monitor.                                   |     |
| 3      | Operator/Process Management | This layer is inside the Kernel, managing processes and threads, especially starting and terminating programs. It interacts with the CPU to order it to take snapshots of the current thread before telling layers 2 and 1 to perform context switching. |     |
| 2      | Memory Management           | Aight, so now this part is the one that manages Page Tables, and works closely withthe  Memory Management Unit. It has a part that handles Page Fault ([[Memory\|Memory]]), and also works closely with Operator/Process Management.                     |     |
| 1      | CPU Scheduling              | Just one step ahead of Hardware, this one basically handles a series of instructions. It just tells the CPU what to execute, manages Process & Thread's lifecycle and runtime                                                                            |     |
| 0      | Hardware                    | Where all operations truly happen                                                                                                                                                                                                                        |     |

> Let's, for better understanding, create a mock map
```pseudocode
1. Your process wants to print to the standard output (stdout), so ```printf()``` which lives on layer 5. Your programm pass the management of your program to the Kernel, and the program is paused.
2. After ```printf()``` is a GNU C library (glibc) wrapper, it calls ```write(stdout, message, strlen(message));```. ```write()``` is a function that's used to... You guessed it, write something. However, you must specify the destination: pipe for IPC, stdout, or a file?
3. After that, the Process Management checks if you have permission to write to this output, since stdout: yes.
4. Now the page table handler (layer 2), It checks where your string lives in Virtual Memory and then maps it to a real Frame. And then the CPU looks at layer 4 to see whether the screen is ready.
5. The layer 4 sends a signal to the monitor, asking whether it's ready or not.
6. During this time, instead of waiting 5 milliseconds, the scheduler assigns another task to the CPU (time-slicing)
7. The monitor directly talks to the CPU, sending an interrupt.
8. The CPU doesn't understand why it's interrupted and visits the Interrupt Vector Table, the table set up by Layers 1 and 2 during booting. It jumps to a specific code there.
9. Layer 1 takes over and sees which pin, and then passes it to the layer 4. Layer 4 sees it as the monitor and sends the signal to layer 3, telling it to handle which program needs output next.
10. The bit is printed to the monitor's memory one by one, instead of waiting for the slow 2 milliseconds delay between typing, the layer 1 (scheduler) assigns another task.
11. Boom, printed to the stdout. What happens next, the success signal streams upwards through the layers until the OS gives control of the program back to your program.
``` 
 
```pseudocode.
1. In a process (The GUI or Terminal), let's use bash. You ask to run an app: Vim
2. The bash calls an API  from the exec family, for example:```execlp()```. It lives on layer 5, the interface.
3. No, because it doesn't interact with the I/O skip layer 4, and goes straight to layer 3. Where it calls the real exec function ```execve()```.
4. Then it calls the layer 2 to wipe all the current data in the Virtual Memory, unallocate the memory in RAM, and create a new Page Table (Text, Data, Heap, Stack segment) so the PID is the same, but the program is different.
5. Boom! There will be no success signal, since the old program is dead.
6. And then you can start coding in Vim without a mouse 🐧.```
```

> So a layer needs to depend on the layer below, but not linearly and definitely; they can skip things out of order. Now let's see what happens when a program goes rogue. A null pointer is a pointer to the address 0 (0x0), it's definitely not something you should touch, so let's see. 

```pseudocode
1. I write a program that dereferences a NULL pointer.
2. The only thing that is fast enough to catch this danger is the MMU.
3. The MMU checks the page table and sees 0x0 in the Virtual Address is not mapped to anything, triggering a page fault.
4. The OS stops your program, specifically the layer 3 order layer 1 to stop it. Then the layer 2 performs inspections and finds out that the address is not yours to begin with.
5. Layer 2 then calls Layer 3 to terminate the program with a SIGSEGV error.
``` 

> Think of it like this, all layer dumps the work on the CPU, and layer 1 is the one managing the super-fast CPU to never be idle, always doing something, never thinking about nothing.

> And as a note, the real Kernel doesn't have physical layers like this; this is just for understanding how the kernel does things. A function may handle processes (Layer 3), check the memory (Layer 2), and manage the CPU (Layer 1). This is an abstraction we use to learn. And trust me, without this I would've screamed 💀🐧. And also, this is my current mental model, feel free to correct me 🐧🐧🐧.

## 3. Massive dumping
> So we know that every request to do something on the hardware must go through a different space and many layers of API, so this is why in real software, we do all the instructions locally first before writing to the disk in one go, so we have to go with the least amount of journey possible.

# Documentation of System Calls in C - Day 2:
> **Shalom!** Aight, I used mental model learning, but time to fix the mess I created first before we move to ```write()``` 🐧

## 1. Hardware has 4 rings, OS uses 2
> So forget about the 6 layers we talked about on Monday, and go back to the number 1, spaces. So in Hardware, there are 4 possible rings; the lower the number, the higher the privilege to the hardware. There are Ring 3 - Ring 0, but Operating Systems only utilize Ring 3 and Ring 0.

## 2. Ring 3 = User Space
> Yep, Ring 3 is basically User Space where every process lives, hence from our last week's mental model, layer 5 - layer 3 live here. It has the least amount of privilege and must go through Ring 0.

## 3. Ring 0 = Kernel
> That's it, Kernel lives at Ring 0 and has unrestricted access to the hardware. Instead, it's a very privileged software (execution environment) that manages the hardware to help you write your code smoothly 🐧👍.

## 4. Fixing misconceptions
> So I wrote layer models; it's a learning abstraction. Real OS doesn't behave that way. 

> In my ```printf()``` example last week, I did some mistakes. ```printf()``` is just a glibc wrapper for system call ```write()``` (Not 1:1, but close enough). So it's basically Ring 3 telling Ring 0 what to do via an API request: that's syscalls, a safe API you can call to send a request to the Kernel, to access restricted hardware actions (writing to disks, requesting more memory, etc)

> Now, for my CPU Scheduler, I made some mistakes as well. It's not a dumping task. CPU doesn't understand processes, it only understands threads (Which in itself is just a series of instructions). Scheduler just decides which threads get the runtime. In Linux, we use the Completely Fair Scheduler (CFS) [[Process & Threads|Process]].

# Documentation of System Calls in C - Day 3:
> Aight, I said syscalls gonna be fine, but it seems I was slapped by it 🐧💀. So today we're gonna be doing 2 things: addressing the mistakes I created, then we're gonna go to some of the most famous syscalls.

## 1. Mistakes
> Aight, let's gonna go through these,

### ```printf()``` debauchery 💀🐧
> Never did I think printing to stdout would be so complicated. So here's the breakdown:

```pseudocode
1. Your process wants to print to the standard output (stdout), so ```printf()``` which lives on layer 5. Your programm pass the management of your program to the Kernel, and the program is paused.
2. After ```printf()``` is a GNU C library (glibc) wrapper, it calls ```write(stdout, message, strlen(message));```. ```write()``` is a function that's used to... You guessed it, write something. However, you must specify the destination: pipe for IPC, stdout, or a file?
3. After that, the Process Management checks if you have permission to write to this output, since stdout: yes.
4. Now the page table handler (layer 2), It checks where your string lives in Virtual Memory and then maps it to a real Frame. And then the CPU looks at layer 4 to see whether the screen is ready.
5. The layer 4 sends a signal to the monitor, asking whether it's ready or not.
6. During this time, instead of waiting 5 milliseconds, the scheduler assigns another task to the CPU (time-slicing)
7. The monitor directly talks to the CPU, sending an interrupt.
8. The CPU doesn't understand why it's interrupted and visits the Interrupt Vector Table, the table set up by Layers 1 and 2 during booting. It jumps to a specific code there.
9. Layer 1 takes over and sees which pin, and then passes it to the layer 4. Layer 4 sees it as the monitor and sends the signal to layer 3, telling it to handle which program needs output next.
10. The bit is printed to the monitor's memory one by one, instead of waiting for the slow 2 milliseconds delay between typing, the layer 1 (scheduler) assigns another task.
11. Boom, printed to the stdout. What happens next, the success signal streams upwards through the layers until the OS gives control of the program back to your program.
```

> So this is wrong, the correct process is:
```pseudocode
1. Your process calls ```printf("Hello")``` 
2. ```printf``` calls ```write(1, "Hello", 5)```
3. Kernel sees one is just a stream of stdout, which is the terminal, so the Kernel puts it into a shared buffer the terminal can access.
4. The terminal is waiting on a ```read()```, then as soon as the data hits the buffer, the terminal converts it to an algorithm: I need to print this word, in this font and size.
5. Now, the terminal needs to send it to the monitor via the GPU, which calls another API called the Graphical API
6. The Graphical API is just a special syscall for the Kernel to pass the instruction to the GPU driver, which is inside the Kernel.
7. The Kernel passes it to the GPU driver
8. The driver converts it into something the GPU can understand
9. The Kernel sends it to the GPU via a buffer, specifically VRAM
10. The GPU constantly monitors this buffer to send it to the Display Engine, which sends it to your monitor, allowing you to finally see "Hello" in your terminal. Pretty complicated, huh? 🐧💀
```

> We ended up learning more about GPU instead of syscalls, huh 💀🐧?

### The CPU actually drops to Ring 0
> So I was wrong, again💀🐧The program stays in Ring 3, the CPU actually descends 🐧💀. So, I finally connected the dots: The CPU has 4 rings of modes, but the OS only operates in Ring 3 for processes and 0 for the kernel. BUT The CPU changes a lot.

> So when your process calls a ```printf()```

```pseudocode
1. ```printf()``` calls ```write()``` which is an syscall.
2. The program calls the ```syscall``` instruction on itself, pausing it. When your computer boots up, your Kernel sets up a backdoor that a process can call, but never change. It immediately tells the CPU to switch to Ring 0 and switch to this specific address to start executing a syscall.
3. Your program never descends; it just freezes, and your CPU switches its mode from Ring 3 to Ring 0, as an analogy: A fighter has a big power, but most of the time he holds back, but when the situation calls for him, he lets out his power. 
4. The Kernel sends the instructions of the syscalls
5. And the CPU begins executing it.
6. The rest are in the ```printf()``` explanation above.
7. Finally, after the fight (request) is over, the fighter (CPU) returns to holding back his power.
```

### A CPU is fast but dumb.
> A bit harsh, I know, but it describes the CPU best: It doesn't understand anything but binaries. Instead, it's the scheduler who makes sure the CPU can do tasks.

> A CPU doesn't understand Process, Threads, Virtual Memory, Virtual Address, anything. It only understands binaries. So your OS creates these abstractions, so you, as a human, can understand the CPU. The CPU is just a very energetic child, and the Kernel needs to make sure he can do all his tasks by "translating" the tasks and then using scheduling tasks to tell it what to execute next.

> MMU manages page tables; it understands what to do, and it walks through the page tables based on the address you asked for, using a specific algorithm. I won't dive deep into this, my head is already hurting 💀🐧

## 2. Famous Syscalls
> I have atoned for my sins 🐧💀 Now time to introduce famous, or rather, frequently used syscalls.

| Syscall                                                                                         | Explanation                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |     |
| ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| ```int open(const char *pathname, int flags, ..., mode_t mode);```                              | This syscall basically tells the Kernel to open a file. Now, a CPU can't talk to the Disk directly, as they live at different speeds. So here's what happened: The Kernel receives ```open()```, the Kernel tells the Disk Controller (In your motherboard, or directly inside the stick for NVMe) to fetch the data, then it puts it in a buffer (A memory in RAM your Kernel and Disk Controller has access to) so the CPU can talk in RAM. This returns the file descriptor (fd), which is just something like a pointer to the file active in RAM (not a memory address). And another gotcha, the OS actually saves the offset of the current reading. |     |
| ```ssize_t read(int fd, void buf[.count], size_t count);```                                     | So now we understand everything is moved to RAM, read becomes easier. It basically means: pass the pointer to the file, pass in where I'll dump it, and tell me how many bytes (characters) you want. But a bit of a gotcha, ```open()``` has something like a cursor, so when you read, the offset. When you call open and read 50 characters, the next time you call 50 characters, it'll be the next 50. It also returns the size of the read bytes, and the offset is pushed based on this number, 0 for End Of File, and -1 for error.                                                                                                                |     |
| ```ssize_t write(int fd, const void buf[.count], size_t count);```                              | So now write, this basically means: give me the pointer to the file, and the thing you wanna dump, either overwriting or appending, depending on how the file was opened with flag, by default it's rewrite everything, but for special cases (```open()``` with ```O_APPEND``` flag, it will be appended instead. And as a gotcha, when you ```write()``` the file on disk hasn't been synced yet, so the OS just tells success so your process can feel fast. A few seconds later (or after close), the OS will flush it to the disk; specifically, the Kernel will take the file in RAM and tell the Disk Controller to write it to the Disk.           |     |
| ```int close(int fd);```                                                                        | The simplest one, huh 🐧💀 This basically closes the file descriptor, and allows it to be used for another file. Return 0 for success and -1 for error.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |     |
| ```int brk(void *addr);```                                                                      | So now brk, as in break, it goes to the end of your heap ([[Memory\|Memory]]), and expands it to the address you give it to (void *addr). It goes to the "break," or rather, the end of the heap, and pushes it further until the address you specify, so you have more memory. 0 for success and -1 for error                                                                                                                                                                                                                                                                                                                                             |     |
| ```void *sbrk(intptr_t increment);```                                                           | Counting manual address is a pain huh?? So this is brk brother, it goes to the current break/line and increments it by how many bytes you requested.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |     |
| ```void *mmap(void addr[.length], size_t length, int prot, int flags, int fd, off_t offset);``` | So... Hello Void Pointer [[Pointer\|Pointer]] 🐧💀. So, as this function does is basically asks the Kernel for a large chunk of memory via anonymous mapping or for a file mapping, instead of copying it directly to RAM, it sets up a page table ([[Memory\|Memory]]) so that many memory can point to the same file in RAM based on page table [[Memory\|Memory]], it's efficient.                                                                                                                                                                                                                                                                      |     |
| ```int munmap(void addr[.length], size_t length);```                                            | Basically, just unmap the address given to you, or unmap the file given to you.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |     |
| ```int unlink(const char *pathname);```                                                         | Ever heard that when you remove a file, it isn't removed? And you just lost access to it, waiting for it to be overwritten by other files? Yep, now you understand ```unlink()``` 💀🐧 But there's a plot twist                                                                                                                                                                                                                                                                                                                                                                                                                                            |     |

## 3. Everything is CRUD... and Everything is a file
> YEP, everything is a file. stdout, stdin, stderr, and socket, GPU, Monitor, and Headset are just files, as long as it does I/O. And since everything is a file, wanna guess? Yep, unified way to access everything. Whether talking to a GPU, process, .txt, or anything, just ```open()```, ```read()```, ```write()```, ```close()``` using their respective file descriptor. How do they achieve this? Virtual File System. When you call ```read()```, the VFS in the Kernel looks at the FD and determines, "Ah, this is a printer," then talks to their driver, and finally the Kernel sends the translated command to the CPU to put to the printer.