---
tags:
  - C
aliases:
  - Pointer
  - Memory
  - Adresss
  - Dangerous
---
# Compilation of my pointer journey
# Documentation of Pointers in C - Day 1:
> A pointer is basically: a variable that holds the address of another variable. We are able to read or modify the value inside the address via dereference.

## 1. What is a pointer
> As I said earlier, a pointer is basically a variable that holds the address of another variable. But before we dive deeper, let's understand RAM, CPU, and what a variable is.
```main.c
#include <stdio.h>

int main(){
	int number = 8;
	int *pNumber = &number; // Store the address of number
	printf("%d\n", *pNumber); // Go to the address of number and see what's inside
	return 0;
}
```

## 2. RAM and CPU
> RAM operates on the same language as the CPU: Binary. It carries a contigous blocks of 0s and 1s (binary), 0 for not being flowed by electricity and 1 for the opposite. However, the CPU interprets them as 1 byte (8 bits), so one memory address (eg, 0xAF) is exactly one byte.

## 3. What is a variable
> A variable is basically a box that acts as its value. So if we have ```int x = 5;``` x behaves as if it is 5. Now, as we talked about, RAM can only store binaries, so how can we have letters like A or B? Simple: American Standard Code for Information Interchange (ASCII). ASCII is a conversion standard in which we can convert a set of numbers to a letter. However, it must be noted that int != equal to char in size. Here is a table for how much a data type takes up space in RAM:

| Type | Memory | Explanation |
|------|-----------|----------------------------|
| Int   | 4 bytes | That's why the maximum value of an integer is a 32-bit number|
| char | 1 byte | Char is just a number converted to a letter via ASCII|
| float | 4 bytes | A float is just bits converted to a float number via the IEEE 754 standard: The first bit determines positive or negative, the exponent determines how much to the right the point is, and the mantissa is just the number.|
| double | 8 bytes | A double is just a float with more precision, however, with double the memory. |
| void | 0 byte | It holds no memory in RAM |
| Pointer | 4 or 8 bytes | Depends on your CPU (32-bit vs 64-bit). It's just a street address |

> For data types > 1 byte, they live contiguously; 4 bytes means 4 contiguous bytes next to each other. And a side note, void cannot be used as a variable type, it can be used as a function return type, and a void pointer, which is a generic placeholder for a pointer.

## 4. Array, String & Struct
> Just as I said before, any type of data lives contiguously next to each other. The same with Array and Struct.

| Type | Explanation |
|-------|---------------|
| Array | An array is a box that holds the same type of variable. All array variables live next to each other. |
| Struct | A struct is just a boxcollection that holds many types of variables. The same with Array, its variables live next to each other|
| String | This is not a native data type; string is just an Array of char |

## 5. Pointer's real usage
> Now, why does all that knowledge about variables matter? There are 2 benefits. The first one is that we know the size of each variable's type. 
1. Passing by reference
> Now, let's say I have this big data:

```main.c
typedef struct{
	char name[100];
	int age;
} User;

int main(){
	User Users[4] = {
		{"Bernardus", 15},
		{"Stanislaus", 15},
		{"Christian", 15},
		{"Anna", 16}
	};
	return 0;
}
```

> If I want to pass it into a function (Pass by value), C would copy the entire value, which is big and costly. Instead, if we accept a pointer, or in C terms, pass by reference, it would be less costly as a pointer to anything is always 4 bytes or 8 bytes:
```main.c
#include <stdio.h>

typedef struct {
    char name[100];
    int age;
} User;

void printData(User *users, int size); // Expect a pointer to a single User (the first one in the array)

int main() {
    User Users[4] = {
        {"Bernardus", 15},
        {"Stanislaus", 15},
        {"Christian", 15},
        {"Anna", 16}
    };

    printData(Users, 4); // Pass 'Users' directly (it automatically decays to a pointer)
    return 0;
}

void printData(User *users, int size) {
    for (int i = 0; i < size; i++) {
        printf("Name: %s\n", users[i].name);
        printf("Age: %d\n", users[i].age);
    }
}
```
- Dynamic memory
> So in C, as we discussed in [[Undefined Behavior]] , in C we manage memory manually, hence to have a variable that we don't know the size of until runtime, we use a pointer:
```main.c
#include <stdio.h> 
#include <stdlib.h> 
#include <string.h> 
int main(int argc, char* argv[]) {
    if (argc < 2) {
        printf("Please provide an argument!\n");
        return 1;
    }

    char *pointer = malloc(strlen(argv[1]) + 1);

    free(pointer); 
    return 0;
}
```
- Accessing the memory of a 'manager.'
> In some cases, like with the FILE type, we actually get the address of a text struct when we open a .txt file:
```main.c
FILE *filePointer = fopen("todo.txt", "a+"); // We gain the address of the file struct.
		if (!filePointer){
			printf("Error opening file");
			return 1;
		};
		createTask(argc, filePointer, argv);
		fclose(filePointer);
```
## 6. &, * and -> symbols
| Symbol | Functionality|
|----------|-------------|
| & | This means, get the address of a variable. |
| * | For initialization ```int *pNumber = &number``` this marks that this variable is a pointer. However, when dereferencing: ```*pNumber```, this means go to the address and get the value. Also, you can change the value when dereferencing: ```*pNumber = 28``` -> This will result in the number's value being changed.|
| -> | This is used for structure access, ```some_struct->value``` is the same as ```(*some_struct).value``` |

# Documentation of Pointers in C - Day 2:
> A change of plan, before we dive into C++ and Rust, we need to understand the power (and danger) of pointers.

## 1. Array decays
> As we talked about in the previous issue, one of the primary usages of a pointer is pass by reference. So, in C, to enable passing an array to a function, the compiler automatically does an implicit conversion from an array to the pointer of the first value. This is useful for passing by reference; however, it results in losing the ability to check the size of the array inside the function.
```main.c
#include <stdio.h>
#include <stdlib.h>

void print_size(int array[]){
	printf("The size of the array: %zu\n", sizeof(array)); // This will always return 4 or 8 depending on the architecture
}

int main() {
	int number[10];
	printf("Size of number array: %zu\n", sizeof(number)); // This will print the sizeof(int) * 10
	print_size(number); /// Will print 4 or 8 no matter what.
	
	int *pNumber = number; // This decays to the address of the first element in the array
	printf("Size of the pointer: %zu\n", sizeof(pNumber)); // This will print 4 or 8
}
```

## 2. Pointer Arithmetic
>The C compiler may optimize our source code into something we don't expect sometimes, but it's actually pretty smart. Pointer arithmetic means that when you do an arithmetic operation (excluding multiplication and division), the pointer doesn't move 1 byte; it moves the size of the value upwards. And from the array decay previously, in real software, people pass the address to the first element via decay, and the size of the array to do pointer arithmetic
```main.c
#include <stdio.h>
#include <stdlib.h>

void loopThroughArray(int array[], size_t size){
	for (int i = 0; i < size / sizeof(*array); i++){ // Use array decay to get the size of each element
		printf("The element %d of the array: %d\n", i + 1, array[i]); // Go up 1 element in the array
		printf("The element %d of the array: %d\n", i + 1, *(array + i)); // It's the same as the above, the compiler converts array[i] -> *(array + 1) = dereference the address of array + i
	}
}

int main() {
	int number[] = {1, 2, 3, 4};
	int *pNumber = number; // Array decays to be the pointer of the first element

	printf("The value of number[0]: %d\n", *pNumber); // Go to the first address (Array decays)
	pNumber++; // Go one size of the type; sizeof(int) == 4 -> Go up 4 bytes
	printf("The value of number[1]: %d\n", *pNumber); // Print 2, the second address

	loopThroughArray(number, sizeof(number));
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/pointer$ gcc -o main main.c && ./main
The value of number[0]: 1
The value of number[1]: 2
The element 1 of the array: 1
The element 1 of the array: 1
The element 2 of the array: 2
The element 2 of the array: 2
The element 3 of the array: 3
The element 3 of the array: 3
The element 4 of the array: 4
The element 4 of the array: 4
```

## 3. String
> As we discussed before, a string is just an array of char. So when we loop through a string, we perform array arithmetic, but with only 1 byte (the sizeof(char)):
```main.c
#include <stdio.h>
#include <stdlib.h>

int main() {
	char name[10] = "Bernardus"; // It's 10 bytes because 9 letters + 1 terminator '\0'
	for (int i = 0; i < sizeof(name); i++){
		printf("%c\n", name[i]); // Go 1 sizeof(char) up which is just 1 byte.
	}
	return 0;
}
```
```bash
Bernardusbernardus@DESKTOP-3HNT5JD:~/projects/experiments/pointer$ gcc -o main main.c && ./main
B
e
r
n
a
r
d
u
s

```

## 4. Stack & Heap - Dangling Pointer
> So Stack and Heap will be discussed later; however, we must know a bit about the difference:

| Memory type | Explanation |
|-------------------|--------------|
| Stack | Stack follows the First in Last out rule. And is limited to one scope only |
| Heap | Heap is manually managed, and can last as long as we need them, but needs to be freed later. |

> The danger arrives from the danger of dangling pointers:

```main.c
#include <stdio.h>
#include <stdlib.h>

int main() {
	int *pX;
	{
		int x = 15; // Lives on the stack, which is only valid in the same scope
		pX = &x; // Get the address of x
	} // x is out of scope, the memory is invalid
	printf("%p", pX); // [[Undefined behavior]], it's unvalid because the address where x used to live has been deallocated
	// but still prints because it hasn't been allocated to another variable
}
```
## 5. Danger of Pointer - double free && invalid pointer ={NULL pointers, dangling pointers}
> Also, the danger of a pointer lies in dangling pointers or pointers to deallocated memory:
```main.c
#include <stdio.h>
#include <stdlib.h>

int main() {
	int *number = malloc(sizeof(int)); // Get the address where the memory is allocated
	*number = 4;
	free(number); // Let go of the access to the memory;
	printf("%d\n", *number); // This will print garbage value -> [[Undefined behavior]]. Because the memory has been deallocated
	number = NULL; // Now the pointer is NULL, no longer pointing to an invalid memory (dangling)
}
```
> NULL is used to remove a dangling pointer, as it makes the pointer point to nothing (Holds no address). However:

```main.c
#include <stdio.h>
#include <stdlib.h>

int main() {
	int *ptr = NULL; // This pointer points to nothing
	printf("%p", ptr); // [[Undefined Behavior]]: NULL points to nothing, hence no address.
	printf("%d", *ptr); // [[Undefined Behavior]]: NULL points to nothing, and there's nothing inside nothing
}
```

> Also, double freeing a memory will lead to [[Undefined Behavior|UB]], or most likely? Segfault:
```main.c
#include <stdio.h>
#include <stdlib.h>

int main() {
	int *number = malloc(sizeof(int)); // Get the address where the memory is allocated
	*number = 4;
	free(number); // Let go of the access to the memory;
	free(number); // [[Undefined Behavior]]: You can't free something that's not yours. Most likely segfault
	printf("%d\n", *number); // This will print garbage value -> [[Undefined behavior]]. Because the memory has been deallocated
	number = NULL; // Now the pointer is NULL, no longer pointing to an invalid memory (dangling)
}
```

## 6. Pointer to a constant and a constant pointer
> A pointer is just a variable that holds the address of a variable. So:

| Name | Syntax | Explanation |
|-------|----------|----------------|
| A pointer to a constant | ```int const *ptr``` or ```const int *ptr``` | This is a pointer to a constant, which means the pointer itself can change the value it holds; however, when derefrencing we can do read-onl,y not chang,e as the variable in which address ptr holds is a constant|
| A constant pointer | ```int *const ptr``` | This is a constant pointer; the variable whose address it holds, the value can change. However, the address of ptr is constant and can't change |
| A constant pointer to a constant | ```const int *const ptr``` | This is the combination of the two above, neither the value of the variable in which the pointer holds can change, nor the address in which the ptr holds | 

``` main.c
#include <stdio.h>
#include <stdlib.h>

int main() {
	const int number = 4;
	const int *pNumber = &number;

	printf("%d", *pNumber);
	// *pNumber = 8; Invalid since pNumber points to a const (immutable)


	// -------------------------------------------------------------------
	int numberTwo = 2;
	int numberThree = 3;
	int *const pNumberTwo = &numberTwo; // This forever points to the number two

	*pNumberTwo = 2 * 2; // Allowed to modify the value of the variable it points to
	// pNumberTwo = &numberThree; Invalid since pNumberTwo is a const pointer (immutable)

	// ----------------------------------------------------------------------
	const numberFour = 4;
	const int *const pNumberFour = &numberFour; // Neither the pointer nor the variable it points to can change
	// *pNumberFour = 8 Invalid since pNumber points to a const (immutable)
	// pNumberFour = &number; Invalid since pNumberTwo is a const pointer (immutable)

	return 0;
}
```
## 7. Void pointer
> Void pointer is a pointer that points to a void type, void type doesn't have memory or bytes. Its primary usage is as a generic placeholder for another type to cast later. We learned about pointer arithmetic and that a pointer is always 4 or 8 bytes. So this is where this type comes in: It is being used as a placeholder you cannot dereference, since we can't go up sizeof(void) bytes, instead later that will be cast or type changed into an appropriate size type. It's used in memory management functions like malloc, free, realloc, calloc, and reallocarray.
```main.c
#include <stdio.h>
#include <stdlib.h>

typedef struct
{
	char name[100];
	int age;
} Person;


int main() {
	Person Bernardusz = {"Bernardusz", 15};
	Person *pBernardusz = malloc(sizeof(Person)); // malloc is implicitly cast by the compiler into a pointer to a Person type
	free(pBernardusz);

	Person *pHuman = (Person*)malloc(sizeof(Person)); // This is the same, but this is explicit
	free(pHuman);
	return 0;
}
```
# Documentation of Pointer in C - Day 3
> **Shalom**! I have ```malloc(sleep_time)``` and ```free(sleep_time)```, Now time to learn what happened to pointers in C++ and Rust

## 1. C++ is an extension of C
> C with classes, now C++ was, in its early day an extension of C. So every valid C program is a valid C++ program; manual pointers still exist in C++.
```main.cpp
#include <iostream>
#include <stdlib.h>
#include <stdio.h>

int main(){
	int number = 7;
	int *pNumber = &number;

	printf("%d\n", *pNumber);
	std::cout << *pNumber << "\n";
	
	// ---------------------------------------------------------------
	
	int *pointer = (int*)malloc(sizeof(int)); // In C++, you must explicitly cast malloc -> your type
	*pointer = 8;

	int *newPointer = new int(8); // In C++, new does 3 things: Allocate memory, immediately place the value there, and returns the address there
	
	printf("The value of pointer: %d\n", *pointer);
	printf("The address of pointer: %p\n", pointer);

	std::cout << "The value of new pointer: " << *newPointer << "\n";
	std::cout << "The address of new pointer: " << newPointer << "\n";

	delete newPointer;
	free(pointer);
	return 0;
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/pointer$ g++ main.cpp -o  main && ./main
7
7
The value of pointer: 8
The address of pointer: 0x636a9a08c6c0
The value of new pointer: 8
The address of new pointer: 0x636a9a08c6e0
```

## 2. RAII & Our new friends: unique_ptr, shared_ptr, and refrence
> I used the ```new``` keyword in the previous example, it does the same thing as malloc but immedately place the value in the allocated memory. But you still need to ```delete```, otherwise [[Undefined Behavior|memory leak]]. Now here's where RAII and out 4 friends enter the scene.
### RAII (Resource Acquisition Is Initialization)
> Before we go deeper, we need to understand that most of the stuff inside the std namespace is functions, constants/values, and **classes**. RAII is basically: Create a pointer in the stack, go to the heap, allocate the memory there, and when it goes out of scope calls the destructor, which deletes the memory on the heap before finally going out of scope. 
```main.cpp
#include <iostream>
#include <memory>

int main(){
	{
		std::unique_ptr<int> p(new int(8)); // Initialize a unique ptr using new. The pointer lives on the stack, and the value on the heap.
	} // Before reaching here, the unique_ptr calls the destructor to remove memory in the heap before going out of scope
	return 0;
}
```
### Unique Pointer - ```std::unique_ptr```
> A unique pointer is an object class that points to an object, and only one value can have one pointer (owner)
```main.cpp
#include <iostream>
#include <memory>
#include <string>
struct Human {
    std::string name;
    int age;

    // Constructor allows make_unique to work
    Human(std::string n, int a) : name(n), age(a) {}
};


int main(){
	std::unique_ptr<Human> Adam(new Human({"The First Human", 15}));
	std::cout << Adam.operator*().name << "\n"; // Go to the address of newHuman, and get the value of name. 
	std::cout << (*Adam).name << "\n"; // unique_ptr is an object, hence you can't directly dereference a pointer, yet this is a case of: operator overloading wrapper.
	//Every time you dereference a pointer, it calls a function that returns the address of the value and dereferences it.
	// This is so that every C program is compatible with a C++ one.
	std::cout << Adam->name << "\n"; // The same as above

	// std::unique_ptr<Human> Steve = Adam; You can't do this: One value can have one owner, you need to: move the ownership
	std::unique_ptr<Human> Steve = std::move(Adam);
	// std::cout << Adam.operator*().name << "\n"; - This is not allowed, [[Undefined behavior]]. Since the value has been moved to Steve.

	std::unique_ptr<Human> Eve = std::make_unique<Human>("The Second Human", 15); // make_unique is the preferred way to make a unique_pointer as it's shorter and safer (Not diving deep)
	return 0;
} // Out of scope, immediately drops (RAII)
```

### Shared Pointer - ```std::shared_ptr```
> Now, if in a unique pointer, one value can only have one owner, a shared pointer, as its name suggests, means one value has many owners.
```main.cpp
#include <memory>
#include <iostream>

int main() {
    std::shared_ptr<int> firstPointer = std::make_shared<int>(100); // Create a shared pointer using the preferred way (Not diving deep into why it's preferred. This is a C series damnit)
    
    std::shared_ptr<int> secondPointer = firstPointer;  // Now p1 


    std::cout << "Count: " << firstPointer.use_count() << std::endl; // Output: 2 - Returns the number of owners a pointers have
    
    std::cout << "Address: " << firstPointer.get() << std::endl; // Prints the address of the firstPointer
	std::cout << "Address: " << secondPointer.get() << std::endl; // Prints the address of the secondPointer

	std::cout << "Value: " << *firstPointer << "\n";
	std::cout << "Value: " << *secondPointer << "\n";
	
    return 0;
} // firstPointer & secondPointer go out of scope, dropped automatically
```
> A bit of a sidenote, shared pointer also uses RAII, but the difference is that there's a counter on the heap. The destructor doesn't mean destroy the value, rather decrement the number of owners sharing the pointer. So until the counter hits 0, the value gonna live on the heap. I'm not gonna deep dive into this, as this is a C series. A C++ RAII can take so long to understand.

### Reference
> Now, C++ reference is similar to that of Rust, but: Reference is basically another alias or nickname to a variable, allowing you to change the value of a variable without poking at memory. Primarily used in pass by refrence function. With one note: A Pointer can be "empty" (NULL). A Reference MUST be initialized when it is created. You can't have int &ref; on its own.

```main.cpp
#include <memory>
#include <iostream>

void modifyValue (int &value, int desiredValue){
	value = desiredValue;
}

int main() {
	int x = 8;
	std::cout << x << "\n";
	modifyValue(x, 123);
	std::cout << x << "\n";
}
```
```bash
bernardus@DESKTOP-3HNT5JD:~/projects/experiments/pointer$ g++ main.cpp -o  main && ./main
8
123
```

# Documentation of Pointer in C - Day 4
> **Shalom**! I have returned from competitive programming hell. Now time to learn Rust 'pointer.'

## 1. Ownership: One value has one owner
> In Rust, one value has one owner. This is implemented to prevent data races, where many pointers can change a single piece of  data.
```main.rs
struct Human{
	title: String,
	age: i8,
}
fn main() {
	let x = 8;
	let y = x; // The Copy Trait: boolean, int, float has Copy trait, so their value won't get moved instead it will be copied;
	println!("The value of x: {x}"); // X is still valid
	println!("The value of y: {y}"); // 

	println!("\n---------------------------------------------------------------\n");

	let adam: Human = Human { title: (String::from("The first Human")), age: (18) };
	println!("Adam's title: {} and his age: {}", adam.title, adam.age);

	let steve = adam; // This time, Adam has String which has no Copy trait, so the ownership is moved

	// println!("Adam's title: {} and his age: {}", Adam.title, Adam.age); Not allowed, the ownership of the value of the struct is moved to Steve.
	println!("Steve's title: {} and his age: {}", steve.title, steve.age);

}
```
> Now we know if it has no Copy trait, the value is moved, we can clone it with the ```.clone()```
```main.rs
#[derive(Clone)] // This allows cloning a complex data structure like Struct
struct Human{
	title: String,
	age: i8,
}
fn main() {
	let adam: Human = Human { title: (String::from("Human")), age: (18) }; // Create a Human named adam with the title "Human"
	let eve: Human = adam.clone(); // Clone the value from Adam.

	println!("Adam is a {} and his age is: {}", adam.title, adam.age); // Adam is valid and living
	println!("And his partner, Eve is a {} and her age is: {}", eve.title, eve.age); // The value of Eve is cloned from Adam
}
```
> As a side note, Rust's String is quite similar to that of C++; it's a pointer to a heap object. So if we have:
```main.rs
fn main(){
	let name: String = String::from("Bernardus"); // This creates an object of String consisting of:
	// A pointer: a String is basically an object with a pointer to the value on the heap
	// The length: Hence my nickname.len()
	// The Capacity: How much a memory was allocated/used for the object
	let nickname = name;
	// println!("Hello my name is: {}", name); Not allowed! If Rust copies the whole object, this object would have the same pointer as name. and when both goas out of scope, the both call free: Classic double free security [[Undefined Behavior|error]]
	println!("Hello my name is: {}", nickname);
	println!("My nickname's address is: {:p}", nickname.as_ptr()); // Get the address of the String value on the heap
	println!("My nickname's length is: {}", nickname.len());
	println!("My nickname's size: {} bytes", nickname.capacity()); // 9 chars = 9 * 1 byte = 9 bytes
	let second_name = nickname.clone(); // This time, it creates a brand new object with a new pointer and a new memory, however the value, length and capacity are copied

}
```

## 2. Rust 'has no real pointers.'
> So in **safe** Rust, there's no way to poke the address, but we have the bread and butter of Rust: Ownership and Borrowing. It's where you reference a variable and then dereference it to either read-only or change the value.
```main.rs

fn main() {
	let number = 54; // Type is inferred, nice.
	let p_number = &number; // pNumber is a 'pointer' or reference to the number.
	println!("The value of number: {}", number);
	println!("The address of number: {:p}", p_number); // Go to the address stored in pNumber (address of number);
	println!("The value of number: {}", p_number); // Now this is nice printing: Go to the address stored in pNumber and see what's inside
	println!("The value of number: {}", *p_number); // Explicit derefrencing

	println!("\n--------------------------------------------------------------\n");

	let second_number = 7;
	let p_second_number = &second_number;	// An immutable read-only reference (You can't change where the reference points to)
	// *p_second_number = 8; Not allowed as it's an immutable reference
	let secp_second_number = &second_number;	// You can have as many immutable references as possible
	let mut thirdp_second_number = &number; // An immutable reference that can change who it points to
	thirdp_second_number = &second_number; // Allowed

	println!("The value of second_number: {}", second_number);
	println!("The address of second_number: {:p}", p_second_number);
	println!("The address of second_number: {:p}", secp_second_number);
	println!("The address of second_number: {:p}", thirdp_second_number);

	println!("\n--------------------------------------------------------------\n");

	let mut third_number = 8;
	let p_third_number = &mut third_number; // It's a mutable reference of value 8 to the p_third_number
	// println!("The value of third_number: {}", third_number); Not allowed!: The value is being borrowed by the p_third_number, so there can only be one mutable reference, and not even the first owner can access it
	// let secp_third_number = &third_number; Now allowed!: Only infinite immutable reference or one mutable reference!
	*p_third_number = 87; // Go to the address of third_number and change what's inside.
	println!("The value of third_number: {}", p_third_number); // The same as above
	println!("The value of third_number: {:p}", p_third_number); // The same as above
	println!("The value of third_number: {}", *p_third_number); // The same as above
	
	println!("The value of third_number: {}", third_number); // Why is this allowed? In older Rust, this would fail. But in modern Rust, the compiler automatically drops borrowing when it's not used anymore
	let secp_third_number = &third_number; // The same as above

}

```
## 3. Pass by reference in Rust 
> So we've seen the word reference over and over, but we haven't tapped into the real usage of pointer: Pass by reference
```main.rs
#[derive(Clone)] // This allows cloning a complex data structure like Struct
struct Human{
	title: String,
	name: String,
	age: i8,
}

fn print_name(human: &Human){
	println!("The title of {} is {}, and his age is: {}", human.name, human.title, human.age);
}

fn change_name(human: &mut Human, name: String){
	human.name = name;
}

fn main() {
	let adam: Human = Human{ name: (String::from("Adam")), title: (String::from("Human")), age: (18) }; // Create a Human named adam with the title "Human"
	let mut eve: Human = adam.clone(); // Clone the value from Adam.

	print_name(&adam); // Pass in an immutable reference to the function
	change_name(&mut eve, String::from("Eve"));
	print_name(&eve);
}
```
## 4. 'Raw' pointer in Rust
> So now we have seen ownership and borrowing, why bother with a pointer? 3 answers: 

| Features | Reasoning |
|-----------|-------------|
| Foreign Function Interface | When your Rust needs to interactwith unsafe languages like C, Rust can't guarantee a C pointer won't be NULL, hence unsafe pointer |
| Kernel | In Rust, you can't guess a memory address, so you need an unsafe pointer |
| Performance | Sometimes the borrow checker is too strict for complex data types, so the developer skips this step using: unsafe |  
```main.rs
fn main() {
    let mut num = 42;

    // CREATING raw pointers is SAFE. 
    // We just cast a reference to a raw pointer using 'as'
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    // println!("{}", *r1); // ERROR: Dereferencing is unsafe!

    unsafe {
        // DEREFERENCING is UNSAFE.
        *r2 = 1337; 
        println!("Value via raw pointer: {}", *r1);
    } 
}
```
# Documentation of Pointer in C - Day 5
> As I said in [[Undefined Behavior]] , we have 4 ways on how a language turns our source code into something that the CPU can execute. And we have covered compiled languages, so time to do purely interpreted, VM-based interpretation with bytecode and JIT. 

## 1. Purely Interpreted
> We have discussed this in our earlier [[Undefined Behavior|UB]] talk, that purely interpreted languages read your source code line by line and tell the CPU what to do. Whenever you assign a variable, the value lives on the interpreter's heap, as your program is inside the interpreter bubble. It's the interpreter's responsibility to call ```malloc```, and ```free``` as the interpreter is usually written in C. Note: What people thought are usually purely interpreted are VM-Based interpreted (Python, Ruby, PHP).

## 2. VM-based interpreted + Bytecode
> So now we're entering the VM-based interpreted like Python and Ruby. There's no pointer in the source code, but behind the scenes, the programming languages use a lot of pointers since the language runtime is written in C (CPython, Ruby)
```main.py

a = 10 # This doesn't create a new box under the hood; it creates a pointer to a pre-existing int object, which is immutable. 
b = a # This creates a pointer to the same object.
b = 8 # This one doesn't change the value, it points to an 8 object.
print(a, b)

# Everything in Python and Ruby is an object. That's why you can call method and operator overloading on them using __functionname__

a.__add__(80) # This returns the value of a + input, which is 80, but doesn't change the value of a
print(a) # the a still points to 10

#--------------------------------------------------------------

x: list = [1, 2, 3] # Create a list/mutable array with the value [1, 2, 3]
y = x # Behind the scene, this creates a pointer to the x, or in other term a refrence to x
y.append(4) # This doesn't just modify the y, since y is just a reference to x, it goes to x and modifies it

print("The value of x: ", x) # [1, 2, 3, 4]
print("The value of y: ", y) # [1, 2, 3, 4]
```

> Now, where does the pointer come in? preallocation, interned strings, and explicit optimization.

```main.py

# Python pre-allocate most used int from -5 to 256
x: int = -5
y: int = -5
print(x is y) # prints true if x has the same address as y
y = x
print(x is y)

a = 1000 # Create a new object 1000
b = 1000 # The Python compiler is smart, and since this is close enough (one file), it just points to the object 1000
c = a # points to the same 1000 object

print(f"Is a the same object as b: {a is b}") # True
print(f"Is a the same object as c: {a is c}\n") # True

# It must be noted that, if the variable is in a different .py file or a library, Python will certainly create a new object

print("-" * 80)

hello = "Hello world" # Creates a new string object with the value of hello world
world = "Hello world" # Interned string: Since this is a simple string and has no special character, the string is interned and points to the same object
print(f"is {hello} the same as {world}: {hello is world}") # True
print(f"The address of hello: {hex(id(hello))} and world: {hex(id(world))}\n") # The same

new = "Hello world!" # This creates a new object string
bro = "Hello world!" # Since "Hello world!" is already created and is a literal and the compiler is smart, it optimizes
print(f"is {new} the same as {bro}: {new is bro}") # bro is a pointer to the same object
print(f"The address of hello: {hex(id(new))} and world: {hex(id(bro))}\n") # The same

string1 = "Hello "
string2 = "world!"
new_runtime = string1 + string2
print(f"is {new_runtime} the same as {new}: {new_runtime is new}") # Now this is a runtime operation, and can't be optimized by the bytecode compiler, so it creates a new object in the heap
print(f"The address of hello: {hex(id(new))} and world: {hex(id(new_runtime))}\n") # False

print("-" * 80)

# There's only one True, Null, and False object in your program, not per file per program, which is a Singleton

boolean_true = True
boolean_true2 = True
print(f"is {boolean_true} the same as {boolean_true2}: {boolean_true is boolean_true2}") # True
print(f"The address of hello: {hex(id(boolean_true))} and world: {hex(id(boolean_true2))}\n") # True

boolean_false = False
boolean_false2 = False
print(f"is {boolean_false} the same as {boolean_false}: {boolean_false is boolean_false2}") # True
print(f"The address of hello: {hex(id(boolean_false))} and world: {hex(id(boolean_false2))}\n") # True

null = None
null2 = None
print(f"is {null} the same as {null2}: {null is null2}") # True
print(f"The address of hello: {hex(id(null))} and world: {hex(id(null2))}\n") # True
```
## 3. Just In Time compiled
> As we talked about in [[Undefined Behavior]]  JIT ~= as VM Based interpretation, with only HotSpot optimization. However, how they manage variables is different. For JIT, not everything is an object: primitives like int, float, char (The one with no capital) are separate, like C/C++/Rust, now the pointer comes in with 
```pointer.java
public class pointer {
    public static void main(String[] args) {
        int x = 8; // For primitives like int, float, char, etc., they don't create a pointer to a pre-allocated value
		int y = 8; // It creates a new variable on the stack
		if (x == y){ // Checks wheter x and y has the same value
			System.out.println("x and y is the same!");
		}
		System.out.println(
			System.identityHashCode(x) + // This autoboxed raw int into an Integer object, which creates a new object on the heap, and after it's used removed the objects 
			" And " + // Just like a VM-based interpreted language like Python
			System.identityHashCode(y) + "\n" // Java has pre-allocated Integer object from -128 to -127. And as a sidenote, this does not return an address, instead a unique Hash code
		);

		Integer a = 1290; // Creates new object on the heap
		Integer b = 1290; // It doesn't optimize like Python; this is a different object
		Integer c = a; // Now this creates a reference to a
		if (a == b){ // For objects, the operator == doesn't check the value, now it checks, does the object live on the same address?
			System.out.println(a + " and " + b + " is the same!");
		}
		else{
			System.out.println(a + " and " + b + " is not the same!");
		}
		if (a == c){
			System.out.println(a + " and " + c + " is the same!");
		}
		System.out.println(
			System.identityHashCode(a) + // Prints the Id of a (Unique)
			", " + 
			System.identityHashCode(b) + // a is not the same as b
			", " + 
			System.identityHashCode(c) + "\n" // c is equal to a, since they're the same object
		);

		c = 1890; // This doesn't change the value of a, instead it creates a new object and points there
		System.out.println("The value of A and C: " + a + ", " + c + "\n");
		// And since a and c is an object, we can call a method with them.

		// Then how do we compare 2 objects' values? ```isequal()```
		if (a.equals(b) && a == b){
			System.out.println("A and B has the same value, but has a different address");
		}
		else if (a.equals(b)){
			System.out.println("A and B has the same value, but has a different address");
		}
		else{
			System.out.println("They're not the same, at all");
		}
	}
}
```

> As a node for JavaScript, for safety reasons, they do not expose functions like ```id``` or ```is``` to get a way to know the memory address. However, among Python, Ruby, and Java, JavaScript (specifically the V8 runtime) is the most aggressive as JS is so dynamic.

```main.ts

// JavaScript treats every number as a float by the IEEE 754 Standard:
let x: number = 8; // This doesn't store the address to the heap, instead it stores the binary of the value
let y: number = x; // This copies the biner to the y variable

y = 9; // do bitwise math to the binary, which is one of the cheapest performance thingsa  CPU can do

console.log(x); // Prints 8
console.log(y); // Prints 9

// The runtime can just checks, wheter the end of the binary is 0 for a value or 1 for an address

// There are 7 primitives in JS: number, string, boolean, bigint, symbol, null, undefined
// And primitives are immutable

let hello: string = "Hello World!" // So we have seen smi for int, but for string we have the Flyweight pattern (The intern table)
let world: string = "Hello World!" // Now for every string, it's put on the intern table, and it doesn't recreate a new primitive; it just points to the same row of the table
// And when you do a runtime operation of concating string, V8 doesn't create a new intern table, it just: Point to String A + B

// The last one is hidden classes and optimization

type addObject = {
	x: number, y: number
}

const add = (obj: addObject) => {return obj.x + obj.y}
let classTest = { // This creates a hidden class for this object: x with id 0 and y id 1
	x: 86, y: 98
}
let secClassTest = { // So next time the V8 doesn't have to check for x, just jumps to the id 0 or 1
	x: 1, y: 100
}
let thirdClassTest = { // This ones create a new hidden class
	y: 190, x: 79
}

console.log(add(classTest));
console.log(add(secClassTest));
console.log(add(thirdClassTest));

// Note there are still pointers to an object on the heap with String, Number, etc types. But most of the time, we use primitives
// So how can primitives have methods?

let myName = "bernardus" // This is a primitive
let uppercaseMyName = myName.toUpperCase(); // For this to work V8 does this:
// It wraps the primitives in an object on the heap
// Performs the method
// Returns the value and immediately removes the object

console.log(myName);
console.log(uppercaseMyName);
```
## 4. Garbage Collector
> VM-Based interpreted and JIT compiled uses the Garbage Collector. In a language where everything is shared, and you don't know when a variable is freed or not, this is where GC comes in. “Most modern VMs use tracing garbage collection, where reachability, not reference counts, determines object lifetime.” So you don't need to call ```malloc``` or ```free``` in a JavaScript or other programs

Chore: Create a video
