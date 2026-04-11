---
banner: "![[download (2).jpeg]]"
---

## The Raw Bytes
> **Shalom!** 🐧 I am free from duties and assignments, now let's go 🐧🐧🐧

### 1. The non Object way of C
> Remember, GNU/Linux is the successor to Unix system. So C works quite the same with Unix: Everything is a file.
> So `open()` and `socket()` still works, it opens the connection and returns a [[System Calls (Syscalls)|File Descriptor]]

```c
#include "spy.h"

long int get_process_ram_usage(int fd);
long int get_total_system_ram(int fd);
void *memory_worker(void *arg);

void get_process_memory(process_data stats[], size_t sizeof_stats){
	int numberOfPID = sizeof_stats / sizeof(process_data);
	
	int ram_fd = open("/proc/meminfo", O_RDONLY);
	long total_system_mem = 0;
	if (ram_fd == -1) {
		fprintf(stderr, "Failed opening /proc/meminfo\n");
	} else {
		total_system_mem = get_total_system_ram(ram_fd);
		close(ram_fd);
	}
	
	for (int i = 0; i < numberOfPID; i++){
		stats[i].total_memory = total_system_mem;
	}
	
	pthread_t workers[numberOfPID];
	for (int i = 0; i < numberOfPID; i++){
		pthread_create(&workers[i], NULL, memory_worker, &stats[i]);
	}
	for (int i = 0; i < numberOfPID; i++){
		pthread_join(workers[i], NULL);
	}
}

long int get_process_ram_usage(int fd){
	char process_buffer[256];
	long int total_memory_usage;
	ssize_t sizeof_process_buffer =  read(fd, process_buffer, sizeof(process_buffer) - 1);
	if (sizeof_process_buffer <= 0) return 0;

	process_buffer[sizeof_process_buffer] = '\0';

	sscanf(process_buffer, "%*d %ld", &total_memory_usage);
	long page_size_kb = sysconf(_SC_PAGESIZE) / 1024; // Get the size of a page ~4KB
	return total_memory_usage * page_size_kb;
}
long int get_total_system_ram(int fd){
	char ram_data_buffer[BUFFER_SIZE];
	ssize_t sizeof_ram_data_buffer =  read(fd, ram_data_buffer, BUFFER_SIZE - 1); // BUFFER_SIZE is 1024 Bytes
	if (sizeof_ram_data_buffer <= 0) return 0;

	ram_data_buffer[sizeof_ram_data_buffer] = '\0';
	
	char *result_ptr = strstr(ram_data_buffer, "MemTotal:");
	long int total_memory;
	if (result_ptr != NULL){
		sscanf(result_ptr, "MemTotal: %ld", &total_memory);
	}
	
	return total_memory;
}

void *memory_worker(void *arg){
	process_data *process_data_struct = (process_data *)arg;
	process_data stackData = *process_data_struct;
	char path[BUFFER_SIZE];
	snprintf(path, BUFFER_SIZE, "/proc/%d/statm", stackData.pid);
	
	int process_fd = open(path, O_RDONLY);
	if (process_fd == -1) {
		fprintf(stderr, "Failed opening process statm for pid %d\n", stackData.pid);
		stackData.memory_usage = 0;
	} else {
		stackData.memory_usage = get_process_ram_usage(process_fd);
		close(process_fd);
	}
	
	if (stackData.total_memory > 0) {
        stackData.memory_percent = ((double)stackData.memory_usage / stackData.total_memory) * 100.0;
    }
	else {
        stackData.memory_percent = 0.0;
    }
	process_data_struct->memory_usage = stackData.memory_usage;
	process_data_struct->memory_percent = stackData.memory_percent;
	process_data_struct->total_memory = stackData.total_memory;
	return NULL;
}
```
Here is the code from my [Proc State CLI](https://github.com/Bernardusz/proc_stat) for a Thread worker. As you can see, everything is a file. We open a `/proc` which in LInux is the System information via `open()`, while `socket()` is specifically for socket. TLDR: File Descriptor is the *Lingua Franca* of LInux (or Operating system).

### 2. The Object Way - Java 🐧
> In Java, it is the same. Java runs on JVM which is a Virtual Machine. SO it resembles closely to real Linux. So Java doesn't change the fundamental. But it adds Object so make the FD consistent.

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;

public class Main {
    public static void main(String[] args) {
		String currentDir = System.getProperty("user.dir");
		System.out.println("Working Directory: " + currentDir);
		try (
			FileInputStream fis = new FileInputStream(currentDir + "/src/Main.java");
			FileOutputStream fos = new FileOutputStream(currentDir + "/src/Main.java", true);
		){
			byte[] b = new byte[fis.available()];
			int n = fis.read(b);
			if  (n != -1) {
				System.out.println(new String(b, 0, n));
			}
			String data = "// New data\n";
			fos.write(data.getBytes());
		}
		catch(Exception e){
			System.out.println(e);
		}
	}
}// New data// New data

```

```bash
import java.io.FileInputStream;
import java.io.FileOutputStream;

public class Main {
    public static void main(String[] args) {
		String currentDir = System.getProperty("user.dir");
		System.out.println("Working Directory: " + currentDir);
		try (
			FileInputStream fis = new FileInputStream(currentDir + "/src/Main.java");
			FileOutputStream fos = new FileOutputStream(currentDir + "/src/Main.java", true);
		){
			byte[] b = new byte[fis.available()];
			int n = fis.read(b);
			if  (n != -1) {
				System.out.println(new String(b, 0, n));
			}
			String data = "// New data\n";
			fos.write(data.getBytes());
		}
		catch(Exception e){
			System.out.println(e);
		}
	}
}// New data
```

> In Java stream is unidirectional data. So every Open (`FileInputStream` or `FileOutputStream`) needs a separate FD for read and write.

> A bit simple, because I am joking with my friends 🐧💀. Tomorrow we got serious.

## Java Way of I/O
> **Shalom**! 🐧And as you can see, I am running out of imagination for titles 💀🐧SO let's just get started.

### 1. Java's Philosophy: Global-attempt
> Java basically bet on the wrong horse, or rather, wrong penguin 🐧💀
 
In the early 90s (when Java was born in 1995), the Unicode Consortium believed 65536 characters would be enough for every language on Earth forever, the same bet IPv4 made.

Sun Microsystems, who build Java, also jumped on this UCS-2 (precursor to UTF-16) because it sounds logical.

Turns out, Ken Thompson (One of the GOAT 🐐 who co-created C and Unix) co-invented UTF-8 in a diner. It won because UTF-8 is the same as ASCII.

Maths, Ancient symbols, and most importantly, Emojis 🐧💀 took over. UTF-16 loses its biggest advantage, and 🐧still takes 4 bytes and 4 UTF-8 chars 💀🐧.

Changing UTF-16 to UTF-8 would break millions or billions of devices. Talk about suffering from your own success 🐧💀

SO as a correction, in [[Collections, Advanced Types & The Memory Cost]] a char is actually 2 bytes in Java

### 2.  From Zero to Hero: Streams vs. Readers
> Bruh 🐧What title did I just write 💀. SO okay, after we translate C's Open/Read/Write/Close, we will see Java way of I/O, specifically files. This is because in WIndows/Linux, Internet and basically everyone follow the UTF-8 conventions, we need a translator.
> In our earlier example we showed using Bytes and translating it to String manually and vice versa for read, due to Java's UTF-16 nature.

```java
import java.io.*;

public class Main {
    public static void main(String[] args) {
		String currentDir = System.getProperty("user.dir");
		System.out.println("Working Directory: " + currentDir);
		try (
			FileInputStream fis = new FileInputStream(currentDir + "/src/Main.java");
			FileOutputStream fos = new FileOutputStream(currentDir + "/src/Main.java", true);
			FileReader fr = new FileReader(currentDir + "/src/Main.java");
			FileWriter fileWriter = new FileWriter(currentDir + "/src/Main.java", true);
			InputStreamReader isr = new InputStreamReader(fis, "UTF-8");
			OutputStreamWriter osw = new OutputStreamWriter(fos, "UTF-8");
			){
			char[] buffer = new char[1024];
			int charsRead;

			// 2. Read into the bucket until the end (-1)
			while ((charsRead = fr.read(buffer)) != -1) {
				// 3. Print exactly what was read
				System.out.print(new String(buffer, 0, charsRead));
			}

			char[] ISRBuffer = new char[1024];
			int charsReadISR;

			while ((charsReadISR = isr.read(ISRBuffer)) != -1) {
				System.out.print(new String(ISRBuffer, 0, charsReadISR));
			}

			fileWriter.write("// New data");
			fileWriter.flush();

			osw.write("// New data");
			osw.flush();
		}
		catch(Exception e){
			System.out.println(e);
		}
	}
}// New data// New data
// New data// New data
```

```bash
import java.io.*;

public class Main {
    public static void main(String[] args) {
		String currentDir = System.getProperty("user.dir");
		System.out.println("Working Directory: " + currentDir);
		try (
			FileInputStream fis = new FileInputStream(currentDir + "/src/Main.java");
			FileOutputStream fos = new FileOutputStream(currentDir + "/src/Main.java", true);
			FileReader fr = new FileReader(currentDir + "/src/Main.java");
			FileWriter fileWriter = new FileWriter(currentDir + "/src/Main.java", true);
			InputStreamReader isr = new InputStreamReader(fis, "UTF-8");
			OutputStreamWriter osw = new OutputStreamWriter(fos, "UTF-8");
			){
			char[] buffer = new char[1024];
			int charsRead;

			// 2. Read into the bucket until the end (-1)
			while ((charsRead = fr.read(buffer)) != -1) {
				// 3. Print exactly what was read
				System.out.print(new String(buffer, 0, charsRead));
			}

			char[] ISRBuffer = new char[1024];
			int charsReadISR;

			while ((charsReadISR = isr.read(ISRBuffer)) != -1) {
				System.out.print(new String(ISRBuffer, 0, charsReadISR));
			}

			fileWriter.write("// New data");
			fileWriter.flush();

			osw.write("// New data");
			osw.flush();
		}
		catch(Exception e){
			System.out.println(e);
		}
	}
}// New data// New data
import java.io.*;

public class Main {
    public static void main(String[] args) {
		String currentDir = System.getProperty("user.dir");
		System.out.println("Working Directory: " + currentDir);
		try (
			FileInputStream fis = new FileInputStream(currentDir + "/src/Main.java");
			FileOutputStream fos = new FileOutputStream(currentDir + "/src/Main.java", true);
			FileReader fr = new FileReader(currentDir + "/src/Main.java");
			FileWriter fileWriter = new FileWriter(currentDir + "/src/Main.java", true);
			InputStreamReader isr = new InputStreamReader(fis, "UTF-8");
			OutputStreamWriter osw = new OutputStreamWriter(fos, "UTF-8");
			){
			char[] buffer = new char[1024];
			int charsRead;

			// 2. Read into the bucket until the end (-1)
			while ((charsRead = fr.read(buffer)) != -1) {
				// 3. Print exactly what was read
				System.out.print(new String(buffer, 0, charsRead));
			}

			char[] ISRBuffer = new char[1024];
			int charsReadISR;

			while ((charsReadISR = isr.read(ISRBuffer)) != -1) {
				System.out.print(new String(ISRBuffer, 0, charsReadISR));
			}

			fileWriter.write("// New data");
			fileWriter.flush();

			osw.write("// New data");
			osw.flush();
		}
		catch(Exception e){
			System.out.println(e);
		}
	}
}// New data// New data
```

`flush()` so Java doesn't hog the data in JVM's RAM and immediately does it.

A simple InputStream, or OutputStream doesn't care about your bytes, it is just a dumb pipe. If you shove UTF-8 to Java's UTF-16 nature, Java will show garbage.

Reader on the other hand converts them directly from UTF-8 to Java's UTF-16. And `InputStreamReader` or `OutputStreamReader` will convert your Input/Output stream to a Reader, which can be applied to a socket.

### 3. The Robust Warehouse - `BufferedReader` & `BufferedWriter`
> The GOAT for reading line.🐐 It is a wrapper for Reader. Everytime you call `.read()` in `FileReader` or `InputStreamReader` they go to the disk. But for `BufferedReader` it grabs a massive chunk once and puts them in RAM. the next chars you just take from RAM.
> While `BufferedWriter` does the same but with writing: Holds everything in one massive chunk before going to the disk to save CPU cycles.

Because it grabs huge chunk of memory, you can see `\n` and `\r`. which brings the legendary `.readLine()` method.

```java
import java.io.*;

public class Main {
    public static void main(String[] args) {
		String currentDir = System.getProperty("user.dir");
		System.out.println("Working Directory: " + currentDir);
		try (
			BufferedReader br = new BufferedReader(new FileReader(currentDir + "/src/Main.java"));
			BufferedWriter bw = new BufferedWriter(new FileWriter(currentDir + "/src/Main.java", true));
		){

			String line;
			// readLine() handles the buffer, the newline searching,
			// and the String creation for you.
			while ((line = br.readLine()) != null) {
				System.out.println(line);
			}

			bw.write("// Hello World");
			bw.newLine();
			bw.flush();

		}
		catch(Exception e){
			System.out.println(e);
		}
	}
}// New data// New data
// New data// New data// Hello World
// Hello World

```

```bash
import java.io.*;

public class Main {
    public static void main(String[] args) {
		String currentDir = System.getProperty("user.dir");
		System.out.println("Working Directory: " + currentDir);
		try (
			BufferedReader br = new BufferedReader(new FileReader(currentDir + "/src/Main.java"));
			BufferedWriter bw = new BufferedWriter(new FileWriter(currentDir + "/src/Main.java", true));
		){

			String line;
			// readLine() handles the buffer, the newline searching,
			// and the String creation for you.
			while ((line = br.readLine()) != null) {
				System.out.println(line);
			}

			bw.write("// Hello World");
			bw.newLine();
			bw.flush();

		}
		catch(Exception e){
			System.out.println(e);
		}
	}
}// New data// New data
// New data// New data// Hello World

```

## NIO - Java I/O Reborn anew 🐧💀
> **Shalom**! 🐧I woke up late today, cause... letś say my alarm didn´t go off... 💀 SO let's finish this ASAP.

### 1. NIO - New I/O 🐧🌟
> Introduced in Java 1.4 and updated in Java 7, it is a new way to do file I/O and takes on a different approach compared to Stream or Reader.

So let's go back to [[System Calls (Syscalls)|System Calls]] as every process, which a JVM is, needs to interact with hardware.

In [[System Calls (Syscalls)|System Calls]] we remember about `mmap()`? In C, it is basically a way to get memory in the heap via system calls. But in Java, it is a bit different.

In Java, we have [[JVM Architecture & WORA |The Heap]], remember? Now there are 2 types of Heap: The "Heap" and Off-Heap/Native Memory.

The Heap is what we learned: JVM/GC manages it. While Off-Heap/Native Memory is managed manually by the programmer/O, S with caveats, which will be explained later.

### 2. Buffer - The Gate Keeper 🐧🛡
> Buffer is a place where you put your data before you write it somewhere else/access it somewhere else.

In I/O, you write it directly to a `byte[]` or copy it from a `BufferedReader` or read line by line with `FileReader`. But in NIO, you use a `ByteBuffer` that comes in 2 flavors:

- `HeapByteBuffer`: IT lives on the JVM Head
	When you eventually tell the OS to write this buffer to disk, the JVM has to copy the data to a temporary hidden "Direct" buffer first. Because GC might move your heap-array mid-syscall.
- DirectByteBuffer`: Lives in Native/Off-Heap memory
	- The JVM basically does C stuff: gives a raw pointer (`void*`). There is no copying. The Kernel writes directly from your RAM to the Disk controller.
	
```java
import java.io.*;
import java.nio.*;
public class Main {
    public static void main(String[] args) {
		ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024);
		directBuffer.put("Hello World".getBytes());
		directBuffer.flip();
		System.out.println(directBuffer);
		directBuffer.clear();
		directBuffer.put("Hello World".getBytes());
		directBuffer.flip();
		System.out.println(directBuffer);
		directBuffer.clear();

	}
}// New data// New data
// New data// New data// Hello World
// Hello World

```

```bash
java.nio.DirectByteBuffer[pos=0 lim=11 cap=1024]
java.nio.DirectByteBuffer[pos=0 lim=11 cap=1024]
```

However, you might get suspicious if DirectByteBuffer lives on the Off-Heap, and is unmanaged by GC, and it was created using mmap, how can we call `munmap()`? The reason is... GC 🐧💀

So when you create a DirectByteBuffer, Java creates a small "Shadow Object" called a `Cleaner`. Both the Buffer and Cleaner live on the heap. 

The Cleaner holds the Deallocater that knows how to call `munmap` or `unsafe.freeMemory`. So the cycle looks like this:

1. Reference Loss: You set your DirectByteBuffer to null.
2. GC Discovery: During the next Garbage Collection cycle, the GC sees that the Buffer object is unreachable.
3. The Phantom Queue: Instead of just deleting the object, the JVM places the associated Cleaner into a special "Reference Queue."
4. The Reference Handler Thread: A high-priority background thread in the JVM is constantly watching this queue. It pulls the Cleaner out and runs its clean() method.
5. The Syscall: The clean() method finally executes the native code that calls munmap() or free() in the OS.

Remember when I said let GC do its thing in [[Collections, Advanced Types & The Memory Cost]]? This time we need to interfere, because GC only triggers when the JVM Heap is full, so if you keep creating a direct buffer while the JVM Heap is empty, eventually the Gate Keeper will call in the final boss of processes: `OutOfMemory` error 🐧💀

So you can use a manual cleaner, which can trigger [[Undefined Behavior|Undefined Behavior]] or worse, Segfault 🐧💀, or use the modern way: arena

```java
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.lang.foreign.*;
//import java.nio.channels.FileChannel;
//import java.nio.file.Path;
import java.nio.file.StandardOpenOption;

public class Main {
    public static void main(String[] args) throws Exception {
        Path path = Path.of("day3_reality.bin");
        long fileSize = 1024;

        // 1. Create your Arena (The Scope)
        try (Arena arena = Arena.ofShared()) {

            // 2. Open a FileChannel first (The C-style fd)
            try (FileChannel channel = FileChannel.open(path,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE,
                StandardOpenOption.CREATE)) {

                // 3. Use the channel to map the memory
                // In JDK 25, map() is a method of FileChannel
                MemorySegment segment = channel.map(
                    FileChannel.MapMode.READ_WRITE,
                    0,
                    fileSize,
                    arena // Pass the arena here to bind the lifecycle
                );

                // 4. Access the memory
                segment.set(ValueLayout.JAVA_INT, 0, 0xDEADC0DE);
                System.out.printf("Wrote Magic: 0x%X%n", segment.get(ValueLayout.JAVA_INT, 0));

            } // FileChannel closes here (Standard OS cleanup)
        } // <--- ARENA CLOSES HERE: munmap() is called IMMEDIATELY

        System.out.println("Memory unmapped. Robustness achieved! 🐧");
    }
}

```

```bash
Wrote Magic: 0xDEADC0DE
Memory unmapped. Robustness achieved! 🐧
```

Arena is basically the boss of memory management. When the Arena goes out of scope, it calls `munmap()` immediately, ensuring memory safety. Basically, [[Memory|RAII]] in practice.

FileChannel will be explained later. Just now, I am using JDK25, and will use the latest official way for NIO.

### 3. Path, Channel, and Segment - The Trio of NIO 🐧🐧🐧
> So in the previous section, we already saw that Arena is the one managing memory lifetime, so now we have **Path**, **Channel**, and **Segment** as the way to do I/O efficiently and safely.

Think of it like this:

| Feature       | What it is in C | Explanation                                                                                 |
| ------------- | --------------- | ------------------------------------------------------------------------------------------- |
| Path          | `char[]`        | It is just an Object holding the path to the file we're dealing with.                       |
| FileChannel   | `open()` -> fd  | It opens the file and is the file descriptor that gives us access to the file. |
| MemorySegment | `void*`         | It calls `mmap()` under the hood, putting the file in RAM, so when you write                |
```java
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.lang.foreign.*;
import java.nio.file.StandardOpenOption;

public class Main {
    public static void main(String[] args) throws Exception {
        Path path = Path.of(System.getProperty("user.dir") + "/src/main/java/main.txt");
        long fileSize = 1024;

        // 1. Create your Arena (The Scope)
        try (Arena arena = Arena.ofShared()) {

            // 2. Open a FileChannel first (The C-style fd)
            try (FileChannel channel = FileChannel.open(path,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE,
                StandardOpenOption.CREATE)) {

                // 3. Use the channel to map the memory
                // In JDK 25, map() is a method of FileChannel
                MemorySegment segment = channel.map(
                    FileChannel.MapMode.READ_WRITE,
                    0,
                    fileSize,
                    arena // Pass the arena here to bind the lifecycle
                );
                String s1 = "Hello World!\n";
                String s2 = "Hello Penguin 🐧";

// Use the byte size of the strings to find the next available slot
                segment.setString(0, s1);
                segment.setString(s1.getBytes().length, s2);
                String text = segment.getString(0L, java.nio.charset.StandardCharsets.UTF_8);

                System.out.println(text);
                channel.truncate(s1.getBytes().length + s2.getBytes().length);
            } // FileChannel closes here (Standard OS cleanup)
        } // <--- ARENA CLOSES HERE: munmap() is called IMMEDIATELY

        System.out.println("Memory unmapped. Robustness achieved! 🐧");
    }
}

```

```bash
Hello World!
Hello Penguin 🐧
Memory unmapped. Robustness achieved! 🐧
```

### 4. Little-Endian & Big-Endian - How do we write a number 🐧❓
> So Endianness is simply the rule that determines the order in which a sequence of bytes is stored in memory for data that has more than a single byte; the CPU has to decide which one goes first.

#### Big-Endian - The Human Order
> It stores the Most Significant Byte at the lowest memory address, the same way human reads number from left to right (10 > 01)

#### Little-Endian - The Hardware Order
> It is basically the reversal of Little-Endian and used by machines because of addition and type casting. Remember Column Addition in primary school? Yep, that's it.

Because in addition, you start with the least significant number, and in other terms, ms, from behind. And CPU is just massive arithmetic. SO Little-Endian becomes standard.

But the Big-Endian becomes standard for internet/ TCP-IP, Thatś why Java provides a way to set what you are typing.

```java
import java.nio.ByteOrder;
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.lang.foreign.*;
import java.nio.file.StandardOpenOption;

public class Main {
    public static void main(String[] args) throws Exception {
        Path path = Path.of(System.getProperty("user.dir") + "/src/main/java/main.txt");
        long fileSize = 1024;

        // 1. Create your Arena (The Scope)
        try (Arena arena = Arena.ofShared()) {

            // 2. Open a FileChannel first (The C-style fd)
            try (FileChannel channel = FileChannel.open(path,
                StandardOpenOption.READ,
                StandardOpenOption.WRITE,
                StandardOpenOption.CREATE)) {

                // 3. Use the channel to map the memory
                // In JDK 25, map() is a method of FileChannel
                MemorySegment segment = channel.map(
                    FileChannel.MapMode.READ_WRITE,
                    0,
                    fileSize,
                    arena // Pass the arena here to bind the lifecycle
                );
                String s1 = "Hello World!\n";
                String s2 = "Hello Penguin 🐧";

// Use the byte size of the strings to find the next available slot
                segment.setString(0, s1);
                segment.setString(s1.getBytes().length, s2);
                String text = segment.getString(0L, java.nio.charset.StandardCharsets.UTF_8);

                System.out.println(text);
                // Force Little-Endian for your .bin file
                var beInt = ValueLayout.JAVA_INT_UNALIGNED.withOrder(ByteOrder.BIG_ENDIAN);
                segment.set(beInt, s1.getBytes().length + s2.getBytes().length, 1024);
                channel.truncate(s1.getBytes().length + s2.getBytes().length + beInt.byteSize());

                // Read it back from the same offset
                int value = segment.get(beInt, 31);
                System.out.println("The value is: " + value); // This will print 1024
            } // FileChannel closes here (Standard OS cleanup)
        } // <--- ARENA CLOSES HERE: munmap() is called IMMEDIATELY

        System.out.println("Memory unmapped. Robustness achieved! 🐧");
    }
}

```
And I just get a revelation: Endianness doesn´t matter in .txt files 🐧💀 I am writing raw bytes to .txt instead of ASCII. Endianess mostly matters in Networking due to Big-Endian being mandatory.
```bash
Hello World!
Hello Penguin 🐧
The value is: 1024
Memory unmapped. Robustness achieved! 🐧
```

### 4. System.out | System.in | System.err - Already opened FD 🚪🐧
> SO remember in [[System Calls (Syscalls)|System Calls]]? `printf()` is just `open()` to an already open file descriptor: 1?
> File Descriptors 0, 1, and 2 are reserved for Stdin, Stdout, and Stderr.
> And Java just wraps them nicely in System.out, System.in, and System.err.

That's it for day 3 🐧
