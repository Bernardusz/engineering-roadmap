---
tags:
  - C
  - error
aliases:
  - Header
  - Error
  - UB
---

# Documentation of header, include, extern, warnings, and sanitizers - Day 1
> **Shalom!** Aight, let's get the wok hot, we're doing this fast.

## 1. How does C -> Binaries
> So we have 2 compilers, GCC and Clang, but what we're gonna be doing is see the requirements of C compliation not how they compile. So we have 4 processes.

### Preprocessing
> So in this step, it doesn't care about the entirety of your code, only specific parts: preprocessor commands (#) and comments (//)

> So anything beginning in # is a preprocessor command, including ```#define```, ```#include```, ```#ifndef```, etc. And for ```#include```, it basically copy paste your imported .h files, you can prove this by doing ```gcc -E main.c``` or ```gcc -E main.c > main.txt```, it will be a big blob of definition before your code is even visible.
>As for comment, that's the opposite, it strips away every command, it deletes them 🐧💀

### Compilation
> Now, this basically turns your preprocessed code into assembly (.s), the last human-readable version of a specific CPU architecture.

### Assembly
> TLDR: Takes .s file and converts it to binaries, resulting in an output (.o) file, but a gotcha, it has "holes" in it.

### Linking
> So now we have our binaries, what's the problem? Simple, you included the definition of the function in your .h file import, but not the function itself. So your binary file has holes because your ```printf()``` function has no implementation. So this phase sees your .o is asking for ```printf()``` and it will search for it, pasting the binary in place.

## 2. Preprocessing Header
> So now we know that everything with # is a preprocessor command, or **preprocessor directive** to be precise. It tells the preprocessor what to do. 

| Directive | Explanation |
|-----------|----------------|
| ```#include``` | Include the content of this file. It doesn't matter if it's h, or c, or cpp, it just performs copy-paste. However, the difference between <> and "" is that <> looks in the system folder (the C compiler files), while "" looks in your folder. |
| ```#define``` | Just performs text substitution, if you ```#define PI 3.14```, the preprocessor will literally replace every instance of PI with the number 3.14 |
| ```#ifdef``` | This is a part of a header guard, ifdef -> if defined, do this. It prevents redefinition, which can lead to errors later. |
| ```#ifndef``` | This is the polar opposites of ```#ifdef```, ifndef -> if not defined, do this. It prevents redefinition, the same as ifdef. |
| ```#endif``` | Basically tells the preprocessor, this line is the end of if |
| ```#undef``` | ```#undef``` basically undefine a macro (a defined using ```#define```) |
| ```#pragma``` | This directive tells the preprocessor to do a special request, either do a guard check like ```#ifdef``` and ```#ifndef``` or something else |
| #error | Tells your compiler to stop compiling with an error message |

## 3. extern - I promise you later
> So extern basically means, "I promise you this exists somewhere else, I'll use it now though." That's it, when you import a global variable through .h, and it will later be combined in the linking stage of compilation.

> The danger? You can't check safety, whether you already declared it or forgot to add the word extern.

```main.c
#include <stdio.h>

int main(){
	extern int x;
	printf("%d\n", x);
	return 0;
}
```
```second.c
int x = 6;
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/header$ gcc main.c second.c && ./a.out
6
```
> You need to compile both, and the linker will combine them accordingly, cause during the linking stage,  they will get combined into a single executable: a.out

# Documentation of header, include, extern, warnings, and sanitizers - Day 2
> **Shalom!**, I got a bunch ton of projects and assignments, let's get this finished. Man, I created this a few days ago, but forgot to comment 🐧💀 🐧💀

## 1. Warnings
> So, as its name says, it's warnings, it warns you of predicted Undefined Behavior ([[Undefined Behavior|Undefined Behavior]]) in your source code before you complete

| Warning | Explanation |
|------------|-------------|
| ```-Wall -Wextra``` | It may say, ```-Wall``` and with extra ```-Wextra```, but it doesn't catch every instance of UB, only the most important one, like reading uninitialized variable, and "valid" human error like ```if (x = 5)```, etc |
| ```-Werror``` | Remember the saying,  "If it compiled, it won't be guaranteed to work,"? This flag is passed to make sure that every warning will throw an error, so it won't compile. |

## 2. Sanitizers
> So Sanitizer basically injects a watcher into your code to make sure your code is "safe." It won't be 100% safe, but it'll catch most of the bugs you'll encounter.

| Sanitizer | Nickname | Explanation |
|-----------|--------------|------------|
| ```-fsanitize=address``` | ASan | If your program ever does anything violating memory, the watcher will immediately crash your code instead of doing UB, and tell you which line messed up. |
| ```-fsanitize=undefined``` | UBSan | Now it does what is named UB. What it does is basically, if your program ever does the spooky stuff in the C standard that "might work on your machine but explode on a server" (like dividing by zero or integer overflows). It'll print a warning to the terminal and keep the program running. But if you want it to terminate your program immediately, add ```-fno-sanitize-recover=undefined```. |

> However, sanitizers inject a watcher into your code, adding RAM and CPU overhead. So only use them in development and use optimization flag once you know your code is safe, like ```-O3``` for the fastest performance. That's it, quite short, huh💀🐧?

# proc_stat

`proc_stat` is a command-line utility written in C that monitors process statistics for one or more specified Process IDs (PIDs). It leverages multi-threading to efficiently collect data in parallel, providing information on process state (e.g., Running, Sleeping) and memory usage (Resident Set Size - RSS), along with its percentage relative to the total system memory.

Link to repository: https://github.com/Bernardusz/proc_stat

## Features

*   **Process State:** Displays the current state of each specified process.
*   **Memory Usage (RSS):** Shows the Resident Set Size (physical RAM usage) for each process in Kilobytes.
*   **Memory Percentage:** Calculates and displays the percentage of total system memory used by each process.
*   **Multi-threaded Data Collection:** Uses POSIX threads (`pthreads`) to fetch process information concurrently, improving performance for multiple PIDs.
*   **Robust Error Handling:** Gracefully handles invalid PIDs or inaccessible process information.

## File Breakdown

Here's a brief overview of what each file in this project does:

*   **`main.c`**:
    *   The entry point of the program.
    *   Parses command-line arguments to extract PIDs.
    *   Initializes an array of `process_data` structures for each PID.
    *   Orchestrates the parallel fetching of process state (`get_process_state`) and memory information (`get_process_memory`).
    *   Prints the collected process statistics in a formatted table, including the total system memory.

*   **`spy.h`**:
    *   A header file that defines the `process_data` structure. This structure holds all relevant information for a process (PID, state, memory usage, total system memory, and memory percentage).
    *   Declares function prototypes for functions implemented in `process.c` and `memory.c`, making them accessible across different source files.
    *   Defines common constants like `BUFFER_SIZE`.
    *   Includes necessary standard libraries for system calls, file operations, and threading.

*   **`process.c`**:
    *   Implements the `get_process_state` function.
    *   Spawns a separate worker thread (`process_worker`) for each PID to read its state from `/proc/<pid>/stat`.
    *   `process_worker` extracts the process state character (e.g., 'R', 'S', 'Z').
    *   Includes robust error handling for cases where process information files cannot be opened or read.

*   **`memory.c`**:
    *   Implements the `get_process_memory` function.
    *   Reads the total system memory once from `/proc/meminfo` and distributes this information to all process data structures.
    *   Spawns a separate worker thread (`memory_worker`) for each PID to read its memory usage from `/proc/<pid>/statm`.
    *   `memory_worker` calculates the Resident Set Size (RSS) and the memory usage percentage based on the total system RAM.
    *   Includes robust error handling for memory information files.

## How to Compile and Run

### Compilation

To compile the `proc_stat` utility, you'll need a C compiler (like GCC) and the `pthread` library.

```bash
gcc main.c process.c memory.c -o proc_stat -pthread
```

### Running the Utility

After successful compilation, you can run the `proc_stat` executable by providing one or more PIDs as command-line arguments.

```bash
./proc_stat <pid1> <pid2> [pid3 ...]
```

**Example:**

To monitor the `init` process (PID 1), your current shell's process (find with `echo $$`), and another common process like `systemd` (often PID 1 if not `init`):

```bash
# Get your current shell's PID
SHELL_PID=$(echo $$)

# Run proc_stat with example PIDs
./proc_stat 1 "$SHELL_PID" 
```

**Finding PIDs:**

You can find the PIDs of running processes using commands like `pgrep` or `ps`:

*   To find the PID of a specific process by name:
    ```bash
    pgrep <process_name>
    ```
    (e.g., `pgrep firefox`)

*   To list all processes with their PIDs:
    ```bash
    ps aux
    ```