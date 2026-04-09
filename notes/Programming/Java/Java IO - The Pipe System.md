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