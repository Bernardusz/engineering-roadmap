---
tags:
  - C
aliases:
  - Undefined Behavior
  - UB
  - Error
  - Memory Leak
---
# Documentation of Undefined Behavior in C - Day 1:

> Undefined Behavior is  when a programmer breaks a set of contracts the compiler sets in place, and if a programmer does break one of the contracts, one of 3 things will happen:

1. OS Stops you (Best Case)
> The program will crash when accessing [[Pointer|memory]] that's not allowed or interfering with another program (Stepping outside the sandbox memory OS gives your program).

2. Random Value
> The program will continue, but the behavior will be unexpected or *undefined*, for example, printing 0.000000, or out of index array with print a garbage value that's inside the sandbox of your program's memory, etc. As long as it doesn't break one of the OS's rules, this will happen

3. Compiler optimization (Worst case)
> C or other compiled languages were designed to run as fast as possible, and the compiler deemed that the programmer is perfect. Hence, they optimize, the if checking may get removed, and if it does happen, it will lead to either OS stopping you (Number 1) or Random Value (Number 2), or a corrupted program, where it may skip function calls, skip if, etc.


So, from this knowledge, we can infer that:

1. Other interpreted languages have some sort of guard rails (Garbage collector, the interpreter itself, or the VM) which make sure no Undefined Behavior can happen. 
2. So in C programming: "Always validate your data BEFORE you use it."
3. Undefined Behavior is random, it is hard to make visible, and it may result in one of the three endings we discussed above

# Documentation of Undefined Behavior in C - Day 2:
> Now, today, we'll learn how Undefined Behavior is prevented with modern languages.

## 1. Purely Interpreted Languages
>This type of language takes your source code, passes it to the interpreter, and then translates the source code line by line, telling the CPU what to do at runtime.  
### Undefined Behavior prevention:
> In purely interpreted languages, the entity managing your variables' [[Pointer|memory]], lifetime, and sets of contract are handled by the interpreter. The interpreter, often written in C, calls functions like malloc and free to manage [[Pointer|memory]] and check your code's safety line by line - enforcing the set of contracts.

## 2. Interpreted + Bytecode + Virtual Machine
> The commonly called 'interpreter' (CPython) actually is a full language implementation that consists of 2 parts: A **Compiler** and a **Virtual Machine**. And inside the Virtual Machine, it consists of an Interpreter of Bytecode, a Garbage collector, etc. This type of language takes your source code and passes it to the front-end compiler, translating the source code to intermediate language (Bytecode). The CPU can't understand this intermediate language, and instead, it is passed to the Virtual Machine. The VM will read the language instruction by instruction using an interpreter inside the VM, managing [[Pointer|memory]] and telling the CPU what to do, and also uses a garbage collector to manage variable [[Pointer|memory]] and lifetime. When the program finishes executing, the used memory is gradually released, leaving a blank spot for other programs to live in later.
### Undefined Behavior prevention:
> In Virtual machine based interpreted languages, the one managing your variables' [[Pointer|memory]], lifetime, and sets of contracts is the Virtual Machine. It manages your variables using the Garbage collector, enforcing the sets of contact. So, before anything is sent to the CPU to execute, the VM checks if your code is 'valid.'

 ##  3. Just In Time Compiled
 Now this type runs nearly the same as Interpreted + Bytecode + Virtual Machine, but the differences lie at runtime. A JIT compiler turns frequently executed parts (hot spots) into native machine code. Combining the best of compiled and interpreted languages.
### Undefined Behavior prevention:
> The same with Virtual machine-based interpreted languages, with caveats: When the hot spots are compiled, the VM also adds a guard to make sure the compiled native code is safe, and if at runtime it turns out the code isn't safe, the VM will de-optimize the native code back to the interpreter mode. 

## 4. Compiled
For compiled languages, they take all your source code, and translate it into machine code (binary) into executeable your machine can run.
### Undefined Behavior prevention:
> For C/C++, they don't have UB prevention. The compilers treat the code and deem you, the programmer, as perfect, and in return, they will perform as fast as possible. They optimize aggressively, sometimes even reorder functions, and remove if statements in the name of 'performance.' Rust is a unique case, but we don't talk about it.

## So, how does OS stop programs from going rogue?
> In the case of Interpreted or JIT languages, every program is contained withing their own [[Pointer|memory]] sandbox, but for 'unsafe' compiled languages that have direct access to the [[Pointer|memory]], the OS stops them using the Memory Management Unit. Whenever a program steps outside of its boundaries, the MMU detects this at the hardware level and then sends a signal to the OS to immediately stop/kill the program before everything else is corrupted.

# Documentation of Undefined Behavior in C - Day 3
> Today, we will learn how Rust achieves both safety and speed.

## 1. Rust is compiled
> Yes, in the previous comment, I said compiled languages like C/C++ don't have UB guard, but Rust is a different case. It is compiled to binary and native code, which the CPU can directly understand, so there is no overhead of an interpreter like in Interpreted or JIT languages. This is the secret to speed.
## 2. Rust manages memory differently.
> Now the reason for safety is that Rust moves checking variables' [[Pointer|memory]] and lifetimes at compile time, resulting in a very long compile time, but checking that your code is safe (No Undefined Behavior, no race conditions, etc), the compiler can optimize aggressively, resulting in a very fast performance.
## 3. Variable's immutability
> Variables in Rust are immutable by default. Resulting in the compiler being allowed to optimize aggressively via:  now having to put the variable in the RAM since the value is known at compile time, either baked into the executable itself or put in the CPU register, which is faster.
## 4. Ownership and Borrowing
> Instead of a raw [[Pointer]] in C, Rust takes a different approach. Rust has a system called Ownership and Borrowing. You can reference a variable in Rust and dereference it to either change or read only. Once you let another variable/owner borrow a value, the first owner cannot be used as the value is being borrowed by the borrower
``` main.rs
fn main() {
	let mut x = 32;
	let	y = &mut x;
	*y = 32;
	// println!("The value of x: {}", x); -> Cannot do, because the value is borrowed by y
	println!("The value of y: {}", y); // This will print the 'user-friendly value. The compiler will follow y to the x and print it.'
	println!("The value of y: {:p}", y);
	println!("The value of y derefrenced: {}", *y);
	// let z = &mut x; -> No can do, Only one mutable borrowing is allowed in one scope

	println!("-----------------------------------------");

	let a =  32;
	let b = &a;
	let c = &a; // For immutable borrowing, as much borrowing as possible is allowed
	println!("The address in which b is pointing: {:p}", b);
	println!("The address in which c is pointing: {:p}", c); // They point to the same address
	println!("The value inside the address in which b is pointing: {}", *b);
	println!("The value inside the address in which c is pointing: {}", *c);

}

```
## 5. Borrow Checker
> The borrow checker's job is to ensure the sets of contracts are fulfilled before the program has finished building during compile time. For example: only one mutable borrowing at one time/scope, etc. This ensures maximum optimization for the final executable. However:
## 6. Rust compiler && Rust panic
> When the borrow checker can't guarantee something is safe (like accessing an array index or integer overflow), the compiler inserts a tiny "Guard" into your binary. And in runtime, if anything crosses this guard, it will throw an unrecoverable runtime error, also known as a Rust panic.

``` main.rs
use std::io;

fn main() {
    let mut max_u32 = u32::MAX;
    let max_i64 = i64::MAX;
	max_u32 += 1; // This will result in a runtime error, also known as "Rust panic" (especially if the compiler can see it coming)
    // Or it might wrap to 0 in release mode.
    println!("Maximum u32 value: {}", max_u32);
    println!("Maximum i64 value: {}", max_i64);

	//-----------------------------------

	let numbers = [10, 20, 30, 40];
    
    let mut input = String::new();
    io::stdin().read_line(&mut input).unwrap();
    let index: usize = input.trim().parse().unwrap(); // if User types num > 3, same, Rust panic

    println!("{}", numbers[index]);
}
```
