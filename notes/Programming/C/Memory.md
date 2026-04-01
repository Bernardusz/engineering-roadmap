---
tags:
  - C
aliases:
  - Memory
  - RAM
  - CPU
  - MMU
---
# Documentation of Memory in C - Day 1
> **Shalom**! Just got out of ILPC comp yesterday, and I am busy as hell this week. So let's get this finished quickly.

## 1. RAM & CPU
> Just as I said in [[Pointer]], RAM is just a giant contiguous 0s and 1s (binaries), and the CPU interprets them by bytes (8 bits/binaries) or multiples of 8. And a fun fact for struct, if the size isn't a multiple of 4 or 8, depending on the architecture, the CPU might give them padding to make the size a multiple of 4 or 8. This was done because the CPU does things faster with the multiple of 8.

## 2. Virtual Memory & Virtual Address
> We already know that RAM is just blocks of 0s and 1s. Then, where do Stack, Heap, and Static come from? Virtual Memory & Memory Layout. Your app doesn't touch real RAM directly; everything always goes through the OS for anything relating to hardware, including software. Virtual Address is a fake address with the maximum of your CPU architecture 2^32 for 32-bit systems, and 2^64 for 64-bit systems. The OS maps Virtual memory to real memory in RAM, but not always. So your program thinks it has up to 0x7FFF12345678, the address doesn't actually exist.

## 3. Pages & MMU
> To avoid having to map every single byte to a virtual memory, which can go up to 2^64, the OS chops memory into chunks, usually 4KB: 4KB of Page Index on Virtual Memory, and Frame (Physical), a fixed-size block of physical RAM (also 4KB).

> When your program asked for an int on the address of ```0x89F1```, it doesn't go directly to RAM; instead, it goes through the Memory Management Unit. It goes to the page table stored in RAM, goes to page number 89, and asks for the offset of 241 (0xF1). The CPU grabs the data (4 bytes) and continues. But to avoid revisiting the Page Table every time, there's the Translation Lookaside Buffer inside the MMU that caches recently visited tables.

## 4. Page Fault & Segfault
> Now we already know Virtual Memory is 'fake' and very optimistic that it can have up to 2^64 Bytes of RAM. But what happened when you access a Virtual Address that hasn't been given a real address in Memory? We trigger a Page Fault.
### Page Fault
> When your program accesses a Virtual Address in Virtual Memory, and it turns out that when the MMU checks the page unmapped or marked invalid, the program is paused by the CPU and gives control to the OS. The OS then checks why. If it is because you just hit the end of your Stack and need more room, the OS allocates more memory for your program and commands the CPU to try again, or:
### Segfault
> You accessed a memory that isn't yours or NULL -> Segfault. 
### Swapping
> But if your RAM is full, the OS will move pages to disk (HDD/SSD). When you try to touch that address, a Page Fault happens, the OS sees the page is on the disk, pauses the program longer, and moves it back to the RAM. This is why programs stutter when memory is low: Pages are moved to disk
## 5. Memory Layout
> We have understood Virtual Memory and Virtual Address, now, where do Stack, Heap, and Static (with Global) come from? Memory Layout in the Virtual Memory. Your RAM only understands binaries; it doesn't differentiate between Stack and Heap. It's an OS thing, a part of the Virtual Memory. Virtual Memory consists of 4 parts:
### Text Segment
> This part of your memory is on the lowest memory possible after the OS gives you. It holds your executable and commands. Read-only to avoid change during runtime.
### Data Segment
> Stores variables that last the entire program's lifetime. Divided into Initialized and Uninitialized (BSS)

| Type | Explanation |
|------|---------------|
| Initialized Data Segment | Data segment that contains global and static variables that have been initialized by the programmer |
| Uninitialized Data Segment/BSS | Stored Uninitialized variables which are automatically set to zero by the system at runtime |
``` main.c
int a; // Uninitialized -> Automatically set to 0 at runtime
int b = 1; // Initialized
int main(){
	static int x; // Uninitialized -> Automatically set to 0 at runtime
	static int y = 2; // Initialized
	return 0;
}
```
### Heap Segment
> This one is used for dynamic memory allocation. It starts at the lowest address possible, right after BSS, and grows upwards. Shared by everyone in the program.
```main.c
#include <stdlib.h>

int main(){
	int *ptr = malloc(sizeof(int)); // Create a pointer to a memory address allocated on the heap
	return 0;
}
```
### Stack Segment
> The stack starts at higher memory addresses and grows opposite to the heap. When Heap and Stack meets what happens?
```main.c
int main(){
	int x = 5;
	return 0;
}
```

> It is structured this way, based on the OS abbility to tamper with it, or in other terms, behavior. The most constant (the ones where the sizes are known at compile time) start at the lowest and grow to the most dynamic. And since the Stack constantly changes more than the Heap (every function parameter, stack/scoped variable, return addresses of functions), it's put on the highest.

## 6. How does a program use RAM and ask for more?
> So we see in Task Manager or systemctl that a program uses 10% of Memory. But we learned that in a Virtual Memory, a program thinks it has up to 2^64 bytes of memory. So, how does a program ask for more Memory? simple: syscalls

```main.c
#include <stdlib.h>

int main(){
	int x = 10; // Create a variable on the stack. The stacks automatically grow down to the heap, up to 8 MB on Linux systems.
	int *ptr = malloc(sizeof(int)); // When we do this, the program asks the OS for memory on the heap.
	// To ask for the memory on the heap, our program (malloc) calls the syscalls brk or mmap.
	// So that's how our program gets memory from the OS.
	return 0;
}
```
### Heap celling
> So, as we know, the Stack starts at the higher addresses. And the heap starts at the lower addresses, and since on 64-bit systems the gap is astronomical (2^64) stack and heap nearly never meet since it's limited by the capability of your RAM. But there's a catch: on 32-bit systems, the gap is only 2^32, which is ~4GB, so it's possible for the stack to meet the heap.

> And as we know from the example above, whenever our program needs more memory and calls ```malloc``` it calls syscall behind the scene to ask the OS to allocate the RAM, then the OS gives the address it allocates, and malloc returns the address, which has already been mapped from Real -> Virtual Address. So what happens when your RAM is full? The OS returns NULL, so that's why ```malloc``` can return NULL. And as a fun fact, if the RAM is truly full, the OS will kill the program with the most RAM usage to save the system. But as a note, on Linux ```malloc``` often succeeds optimistically, but Out Of Memory failure may happen later.

### Stack Overflow
> In my code above, we see the stack can grow up to 8MB, why so small? Because the stack is used for local/scope variables, function parameters, and return addresses for each function call, it's extremely tiny. The 8MB limit is actually to keep you safe, if a recursive function like this:
```main.c
#include <stdlib.h>

void foo(){
	foo(); // This will cause a stack overflow because each function call uses stack space.
}
int main(){
	foo();
	return 0;
}
```
> That never stops, you can't have the time to kill the program before it hogs all your RAM, that's why the limit is imposed, cause most of the time anything > 8 MB on the stack is a bug or undefined behavior, as we discussed in [[Undefined Behavior]] .

Main reference: https://www.geeksforgeeks.org/c/memory-layout-of-c-program/

# Documentation of Memory in C - Day 2
> So we have learned about Stack, Heap, Static, and Global, now where do variables live? Let's get this quick, since tomorrow I'll be going out on a retreat.

## 1. C - The bare metal
> So in the C programming language, the distinction between stack, heap, static, and global is very clear:
```main.c
#include <stdlib.h>

// int *globalPtr = malloc(sizeof(int)); Not allowed, static/global variables live on the Data Segment, not heap.
int *globalPtr; // This is allowed, we can create a pointer variable in global
int main(){
	globalPtr = malloc(sizeof(int)); // Now the global pointer points to something on heap.

	int stackVariable = 10; // This is a stack variable, because it's created inside a scoped function (main)
	
	int *heapVariable = malloc(sizeof(int)); // Dynamic memory allocation on heap
	*heapVariable = 90; // Assign value to the memory location pointed by ptr.
	free(heapVariable); // Free the dynamically allocated memory.

	// static int y = malloc(sizeof(int)); Not allowed, static or global variables cannot be initialized with dynamic memory allocation. It's because the value lives on the Data segment, not the Heap segment.
	static int staticVariable = 20; // This is a static variable, it lives on Data Segment.
	// static int *staticPtr = malloc(sizeof(int)); This is not allowed, the data lives on the Data Segment, not the Heap.
	static int *staticPtr; // This is allowed, we can create a pointer variable in static.
	staticPtr = malloc(sizeof(int)); // Now the static pointer points to something on heap.
	return 0;
}
```
### Memory Padding
> So we already learned about memory, and the CPU doing things faster with the multiple of 4 or 8. That happens to struct as well
```main.c
#include <stdlib.h>
#include <stdio.h>

typedef struct{
	char a; // 1 byte
	int id; // 4 bytes
	char b; // 1 byte
} Data; // Why is this struct padding to 12 bytes instead of 6 bytes?
// It's because of memory alignment and padding added by the compiler for efficient access.
// The CPU accesses memory in chunks, hence 12 bytes. The CPU aligns the data with the largest size on the struct.
// To align the 'int id' member to a 4-byte boundary, the compiler adds 3 bytes of padding after 'char a'. If the largest member is a double (8 bytes), it would align to 8 bytes.
// Similarly, it adds 3 bytes of padding after 'char b' to make the total size a multiple of 4 bytes.

typedef struct{
	char a; // 1 byte
	char b; // 1 byte
	int id; // 4 bytes
}goodData; // Now why is this struct only 8 bytes?
// By rearranging the members, we minimize padding.
// 'char a' and 'char b' take 2 bytes, followed by 2 bytes of padding to align 'int id' to a 4-byte boundary.
// This results in a total size of 8 bytes.

typedef struct {
	double d;  // 8 bytes: The largest member is double, so alignment is to 8 bytes.
	char b;    // 1 byte
	char a;    // 1 byte
	// 6 bytes of padding added here to make the total size a multiple of 8 bytes (16 bytes).
} doubleData;


int main(){
	printf("Size of Data struct: %zu bytes\n", sizeof(Data)); // Outputs 12 bytes due to padding.
	printf("Size of goodData struct: %zu bytes\n", sizeof(goodData)); // Outputs 8 bytes with minimized padding.
	printf("Size of doubleData struct: %zu bytes\n", sizeof(doubleData)); // Outputs 16 bytes due to alignment to 8 bytes.	
	return 0;
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/memory$ gcc -o main main.c && ./main
Size of Data struct: 12 bytes
Size of goodData struct: 8 bytes
Size of doubleData struct: 16 bytes
```
## 2. The C++ RAII and Rust - A stack object -> Heap value
> Now we know that from C, where variables live, now how about one step of abstraction? We enter the C++ and Rust lane. They have totally different philosopy but their approach to where variables live is similar.
```main.cpp
#include <iostream>
#include <string>
#include <memory>
int *globalPtr = new int(6); // This is allowed because it does the same thing as staticPtr inside the main function below.:
int globalVariable = 6; // Assign value to the memory location pointed by globalPtr.

int main(){
	int stackVariable = 6; // This is still a stack variable.

	int *heapVariable = new int; // Dynamic memory allocation on heap using C++ new operator, replacing malloc.
	*heapVariable = 15; // Assign value to the memory location pointed by ptr.
	delete heapVariable; // Free the dynamically allocated memory using C++ delete operator.
	
	static int staticVariable = 30; // This is a static variable, it lives on Data Segment.
	static int *staticPtr; // This is allowed, we can create a pointer variable in static.
	staticPtr = new int; // Now the static pointer points to something on the heap using the C++ new operator.
	*staticPtr = 45; // Assign value to the memory location pointed by staticPtr.
	delete staticPtr; // Free the dynamically allocated memory using C++ delete operator.

	static int *staticPtr = new int(6); // This is allowed because new does a few things behind the scenes:
	// it reserves a memory block on the Data Segment for the pointer variable staticPtr,
	// It creates a secret startup function for global that runs before main(), but for static, it does the same thing as the earlier example
	// then it reserves a memory block on the Heap Segment for an integer and initializes it with the value 6,
	// finally, it makes staticPtr point to that memory block on the Heap Segment.

	// The only difference is where the heap memory allocation happens, either after main() runs for the first example and before for the second example.

	std::string str = "Hello, World!"; // Now for String, this is where things get interesting.
	// String is an object, created on the stack, but internally, it manages a dynamic array on the heap to store the actual characters of the string.
	// Don't forget a string is just an array of characters.
	// This allows strings to be flexible in size while still providing the convenience of stack allocation for the string object itself.

	// This is the same case with all std:: objects in C++, like std::vector, std::map, etc. They're all stack objects that manage heap memory internally for their data storage.
	std::unique_ptr<int> uniquePtr = std::make_unique<int>(42); // Unique pointer managing heap memory.
	std::shared_ptr<int> sharedPtr = std::make_shared<int>(84); // Shared pointer managing heap memory.
	// The object constructors allocate memory on the heap and initialize the values.
	// No need to call free, the destructors of uniquePtr and sharedPtr automatically free the heap memory when they go out of scope.
	// This is Resource Acquisition Is Initialization (RAII) in C++.

	// This is why I said C++ is similar to Rust in terms of memory management, because of these smart pointers.

	return 0;
}

void foo(){
	static int functionStaticVar = 50; // C++ saves a secret boolean value behind the scene:
	// If the boolean is false (Usually the first time the function is called), C++ would initialize the variable and toggle the boolean to true to avoid memory allocation multiple times.
}
```

> Now what about Rust?

```main.rs
struct Point {
	x: char,
	y: i64,
} // Now Rust follows the same padding rules as C/C++ for structs.
// However, Rust can reorder struct fields to minimize padding and optimize memory layout.

static globalVariable: i32 = 42; // This is a global variable, stored in the data segment.
static mut mutableGlobalVariable: i32 = 65; // This is a mutable global variable, stored in the data segment. Needs Mutex for safe access.
// mutableGlobalVariable = 75; Now allowed: Data lives on the Data Segment. Modifying the mutable global variable is unsafe.
const _CONSTANT_VARIABLE: i32 = 100; // This is a constant, instead of living on the Data Segment, it's more like copy-paste, wherever you use this variable, the compiler changes it to the value (100).
fn main() {
    println!("Hello, world!");
	let stackVariable: i32 = 5; // This is a stack variable

	let heapVariable: Box<i32> = Box::new(10); // This is a stack object pointing to a heap value, allocated using Box. The same as a pointer.
	let heapString: String = String::from("Hello, Heap!"); // String is a stack object pointing to an array of characters on the heap, the same as C++ std::string.
	println!("x: {}, y: {}", stackVariable, *heapVariable);
} // y is dropped here, and the heap memory is freed automatically because of the Destructor.
```

## 3. VM-Based Interpreted - Everything is an object
> So in the pointer section, we learned that everything in VM-Based interpreted like Python, is just a pointer to an object on the heap. Not a stack object pointing to a heap value, but **a stack reference/pointer pointing to an object on the heap**.
```main.py
intObject: int = 8 # int Object is a pointer/reference to a PyObject on the heap memory
stringObject: str = "Hello, World!" # str Object is a pointer/reference to a PyObject on the heap memory, the same as intObject

listObject: list[int] = [1, 2, 3, 4, 5] # listObject is a pointer/reference to a PyObject on the heap memory, the same as intObject and strObject
referenceListObject: list[int] = listObject # referenceListObject is a pointer/reference to the same PyObject as listObject
print(f"intObject: {intObject}, id: {hex(id(intObject))}")
print(f"stringObject: {stringObject}, id: {hex(id(stringObject))}")
print(f"listObject: {listObject}, id: {hex(id(listObject))}")
print(f"refrenceListObject: {referenceListObject}, id: {hex(id(referenceListObject))}")
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/memory$ python main.py
intObject: 8, id: 0xb36188
stringObject: Hello, World!, id: 0x7b153b73b8f0
listObject: [1, 2, 3, 4, 5], id: 0x7b153bad4b40
refrenceListObject: [1, 2, 3, 4, 5], id: 0x7b153bad4b40
```

## 4. JIT Compiled - Half Object, Half Primitives
> So now we're entering Java and JavaScript land. And they actually behave similiarly or almost identical to C++ and Rust, just without the explicit memory management, since they're handled by Garbage Collector.
```memory.java
public class memory {
	public static void main(String[] args) {
		int stackVariable = 8; // stackVariable is a primitive, stored in the stack memory
		Integer intObject = Integer.valueOf(10); // int Object is a pointer/reference to an Object on the heap memory
		String stringObject = "Hello, World!"; //stringObject is a pointer/reference to an object on the heap memory, the same as intObject

		int[] listObject = {1, 2, 3, 4, 5}; // listObject is a still an object, stored on the heap memory, the variable is a refrence/pointer to that Object
		int[] referenceListObject = listObject; // referenceListObject is a pointer/reference to the same Object as listObject
		System.out.println("stackVariable: " + stackVariable + ", id: " + Integer.toHexString(System.identityHashCode(stackVariable))); // Temporary turned an int to an Integer object for identityHashCode (Autoboxing).
		System.out.println("intObject: " + intObject + ", id: " + Integer.toHexString(System.identityHashCode(intObject))); // 
		System.out.println("stringObject: " + stringObject + ", id: " + Integer.toHexString(System.identityHashCode(stringObject)));
		System.out.println("listObject: " + java.util.Arrays.toString(listObject) + ", id: " + Integer.toHexString(System.identityHashCode(listObject)));
		System.out.println("refrenceListObject: " + java.util.Arrays.toString(referenceListObject) + ", id: " + Integer.toHexString(System.identityHashCode(referenceListObject)));
	}
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/memory$ java memory.java
stackVariable: 8, id: 341b80b2
intObject: 10, id: 2b546384
stringObject: Hello, World!, id: 3c87521
listObject: [1, 2, 3, 4, 5], id: 2aece37d
refrenceListObject: [1, 2, 3, 4, 5], id: 2aece37d
```
```main.ts
let stackVariable: number = 42; // A primitive number, a stack value
let stringObject: string = "Hello, World!"; // Despite string being called a primitive, it behaves like an object in memory. 
// Primitives just means they are immutable. Strings are stored in heap memory. When you edit a string, you replace the whole string in memory.

let arrayObject: number[] = [1, 2, 3, 4, 5]; // Arrays are objects stored in heap memory
let referenceArrayObject: number[] = arrayObject; // referenceArrayObject points to the same array in heap memory as arrayObject
arrayObject.push(6); // Modifying arrayObject will also affect referenceArrayObject since they point to the same object

console.log(`arrayObject: ${arrayObject}, referenceArrayObject: ${referenceArrayObject}`); // Print arrayObject and its reference
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/memory$ node main.ts
arrayObject: 1,2,3,4,5,6, referenceArrayObject: 1,2,3,4,5,6
```

# Documentation of Memory in C - Day 2.5
> Chop, chop, chop. Let's get this done, I want to play Honkai Impact 3rd before my retreat 🐧💀 We'll prove the structure of memory layout: Text Segment, Data Segment, Heap Segment, Stack Segment.

```main.c
#include <stdlib.h>
#include <stdio.h>

// int *globalPtr = malloc(sizeof(int)); Not allowed, static/global variables live on the Data Segment, not heap.
int *globalPtr; // This is allowed, we can create a pointer variable in global
int globalVariable = 6; // Create a global variable in Data Segment.
int main(){
	globalPtr = malloc(sizeof(int)); // Now the global pointer points to something on heap.
	*globalPtr = 80; // Assign value to the memory location pointed by globalPtr.
	int stackVariable = 10; // This is a stack variable, because it's created inside a scoped function (main)
	
	int *heapVariable = malloc(sizeof(int)); // Dynamic memory allocation on heap, but heapVariable itself is on the stack.
	// This is why memory leak happens, because when the pointer goes out of scope and we forget to free, the memory on the heap is orphaned, and directly ties to my Undefined Behavior documentation.
	*heapVariable = 90; // Assign value to the memory location pointed by ptr.

	// static int y = malloc(sizeof(int)); Not allowed, static or global variables cannot be initialized with dynamic memory allocation. It's because the value lives on the Data segment, not the Heap segment.
	static int staticVariable = 20; // This is a static variable, it lives on Data Segment.
	// static int *staticPtr = malloc(sizeof(int)); This is not allowed, the data lives on the Data Segment, not the Heap.
	static int *staticPtr; // This is allowed, we can create a pointer variable in static.
	staticPtr = malloc(sizeof(int)); // Now the static pointer points to something on heap.
	*staticPtr = 40; // Assign value to the memory location pointed by staticPtr.
	
	printf("The value of staticVariable: %d, The Address of staticVariable: %p\n", staticVariable, &staticVariable); // Static, Global variables should have the lowest address in memory.
	printf("The value of globalVariable: %d, The Address of globalVariable: %p\n", globalVariable, &globalVariable); // As they're located in the Data Segment, the lowest segment before the text segment.
	printf("The value of staticPtr: %d, The Address of staticPtr: %p, and the address stored by staticPtr: %p\n", *staticPtr, &staticPtr, staticPtr); // Now this is a unique case. We have 3 values:
	printf("The value of globalPtr: %d, The Address of globalPtr: %p, and the address stored by globalPtr: %p\n", *globalPtr, &globalPtr, globalPtr); // The address of the pointer (static/global), the address stored by the pointer (heap), and the value at the address stored by the pointer, and the value inside the pointer
	printf("The value of heapVariable: %d, The Address of heapVariable: %p\n", *heapVariable, heapVariable); // Heap variables should have the second lowest address in memory. Right after the Data Segment.
	printf("The value of stackVariable: %d, The Address of stackVariable: %p\n", stackVariable, &stackVariable); // And stack have the highest address in memory.

	free(heapVariable); // Free the dynamically allocated memory.
	free(staticPtr); // Free the dynamically allocated memory.
	free(globalPtr); // Free the dynamically allocated memory.
	return 0;
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/memory$ gcc -o main main.c && ./main
The value of staticVariable: 20, The Address of staticVariable: 0x5c1b39126014
The value of globalVariable: 6, The Address of globalVariable: 0x5c1b39126010
The value of staticPtr: 40, The Address of staticPtr: 0x5c1b39126028, and the address stored by staticPtr: 0x5c1b61f9d2e0
The value of globalPtr: 80, The Address of globalPtr: 0x5c1b39126020, and the address stored by globalPtr: 0x5c1b61f9d2a0
The value of heapVariable: 90, The Address of heapVariable: 0x5c1b61f9d2c0
The value of stackVariable: 10, The Address of stackVariable: 0x7ffceff0d5ac
```
> Now let's investigate:

| Variable | Where it lives | Adress | Explanation |
|---------|----------------|-------------|-----------|
| staticVariable | Data segment | ```0x5c1b39126014``` | It is the second lowest address on the list, to be expected since it lives on the Data segment |
| globalVariable | Data segment | ```0x5c1b39126010``` | The lowest address on the list. |
| &staticPtr | Data segment | ```0x5c1b39126028``` | This is the address of the static pointer on the Data segment, not the pointer. |
| &globalPtr | Data segment | ```0x5c1b39126020``` | Address of the global pointer variable itself, stored in the data segment |
| heapVariable | Heap | ```0x5c1b61f9d2c0``` | Now we're entering the Heap, this variable lives on the stack, but the pointer points to a heap address, which usually is the second lowest after static and global |
| staticPtr | Heap | ```0x5c1b61f9d2e0``` | This is the address that &staticPtr is pointing to; it lives on the heap, the memory allocated by the OS |
| globalPtr | Heap | ```0x5c1b61f9d2a0``` | The same case as staticPtr |
| stackVariable | Stack | ```0x7ffceff0d5ac``` | A massive leap from the highest Heap address, as expected since Stack starts at a higher/highest possible memory |

> We have proven the relative ordering of the memory layout structure! I printed all the variables on each segment, and we proved the structure is: Text Segment (I didn't print it, since it's the executable of your program) -> Data Segment -> Heap Segment -> Stack Segment.