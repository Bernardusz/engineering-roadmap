---
banner: "![[download (2).jpeg]]"
---
# Documentation of JVM Architecture & Magic Behind WORA - Day 1
> YEP, I'm on a midterm exam, that's why I haven't done anything lately, and I got Civic study tomorrow, but Java is the way to go 💀🐧 So let's get this quick before Indonesian slaps me tomorrow 🐧

## 1. The Magic of "Write Once, Run Anywhere"

> So, what do you think about Java? Enterprise? Stable? Verbose 💀? The interesting part is that any Java code can run "Anywhere." How? Java Virtual Machine,
> If in C or other compiled languages, you get an executable like .out or .exe/.deb, which won't work in Linux if you compile it for Windows. In Java, you don't get those.

## 2. Java & OpenJDK
> There are languages, and there are compilers and runtimes. You have the C language, and `gcc` & `clang`. For Java, there are similar instances for the JIT.

| Part of Java ecosystem | Type | Explanation                                                                                                                                                                                                                                                                                                                                                                                          |
|--------------------------|-------|--------------|
| OpenJDK | Toolchain | The same as you installing `build-essentials` for C, in Java, installing OpenJDK is equal to installing all the stuff you need for Java development (minus the IDE of course 🐧). OpenJDK is just one of many Toolchain implementations; there are Oracle's JDK, Amazon's JDK, etc                                                                                                                   |
| `javac` | Compiler | Yep, compiler. But isn't Java JIT? Remember when I talked about UB prevention [[Undefined Behavior\|Undefined Behavior]]? Yep, Java compiles your source code into intermidiate language called `.class`. This is the equivalent of ```gcc -o main main.c```.                                                                                                                                        |
| `java` | Launcher | The Launcher, so in C you call ```./a.out``` or ```./main```. In Java, after you compile `.java` -> `.class`, you call ```java Main``` without `.class` to launch a JVM process & and load it into memory. A plot twist in Java 11 allows you to run ```java Main.java``` immediately, this is the same as ```gcc -o main main.c && ./main``` but the .class is stored in memory, not disk. |
| JVM | Virtual Machine | This deserves its own section, but TLDR: JVM is like the translator/interpreter (JIT) of Java. It takes `.class` files and translates them into machine instructions, simultaneously handling memory (GC ) and pointers, etc                                                                                                                                                                         |
| Maven & Gradle | Build Tools | It's just as painful to type many `.c` files to `gcc` as it is with Java. So basically Maven & Gradle are just build tools for Java that handle a big folder structure.                                                                                                                                                                                                                              |

## 3. Maven & Gradle

> So, you're tired of entering every `.c` file to `gcc` every time you change something? Yep, that pain also exists in Java. Luckily, we have Maven & Gradle.
> Their functions are the same: compile all `.java` source code, link `.class` files, and run them. You just write the coordinate (Group, Artifact, Version) in a file:

- Maven: Uses XML (`pom.xml`), very rigid & stable, labeled the 'old way.'
- Gradle: Uses a script (Groovy or Kotlin: `build.gradle`). Faster & more flexible. The new default.
> However, they also help you install dependency as in Java, you don't have a universal package manager like `npm`. When you run mvn install or `./gradlew build`, the tool reaches out to a server (Maven Central), downloads the `.jar` files you need, puts them in a local cache, and links them.

## 4. JVM Internals
> So, an analogy here to help it stick faster. Java apps are like Electron. When you have an Electron app, you bundle Chromium and web technologies for each app, the sam with Java and JVM.
> However, it's not as severe as Electron, as the JVM is already very optimized, and the JVM gets stripped down to the absolute bare minimum for your app's needs.

### JVM Memory Management - Garbage Collector

> Welcome back, Virtual Memory & Virtual Address ([[Memory|Memory]]) 🐧💀 So each JVM process is the same as an OS process, with the difference being that inside that process, it has a JVM. So every Java app is just an instance of JVM, and .class files are data on the process's heap, being translated by JVM in real time on the fly, except for hotspots.
> JVM process's internal is divided into two specific zones for the process run by JVM (.class translated by JVM), the same as an OS process, Heap, Stack, and The Method Area (The same as .`text` and `.data` segments.

#### The Heap

> Welcome back Heap 💀 Yep, basically inside a process inside JVM, you'd have a heap, everything created with `new` goes here,'
> Just remember that this is the Heap of the process inside the JVM, not the JVM process's heap.
> Remember, JVM is Java Virtual Machine, it's a small OS Process, inside JVM runs your process (GC, Interpreter, JIT, your .class files), the memory of your own process (.class files) is being translated directly to real OS memory by the JVM
> And speaking of GC, you don't do pointers in Java, ```free()``` or ```malloc()```, It''s the Garbage Collector's job to do so. It's a background thread that cleans unused variables and dead objects.
> Remember Mutex ([[Process & Threads|Process]]) ? That's basically it, to avoid removing data which is still used, there's an instance of Stop-The-World, so basically to move objects and updates refrence, all other threads will be stopped first.
> However, it's not a single global mutex: for most objects that die young, it is placed in Younge Generation. The GC cleans this area frequently and barely noticeable by human eyes (a few milliseconds)
> However, if an Objects survive several cleans, it is promoted into the Old Generation. If this gets full, the GC will perform a full GC, which create ssttuters.

#### The JVM Stack

> The same as each Thread having its own stack, it's the same as it is in Java. No explanation needed [[Process & Threads|Process]] 

#### The Method Area

> Sounds fancy, until you know it's just the same as Text and Data segments, it stores class structures, method code, and `static` variables.

### JVM Just In Time Compiler

> This is the translator inside the JVM, it has 2 gears:

#### Interpreter

> Starts immediately. Translates bytecode to machine code line-by-line. The same as Interpreter in Python and Ruby ([[Undefined Behavior|Undefined Behavior]])

#### JIT Compiler (Just-In-Time)

> Now, we get to the part where Java is efficient and faster than VM-Based Interpreted languages, JIT Compilation.
> As its name suggests: Just-In-Time, As your Interpreter runs, JIT watches, are there any sections/functions running many times, then if there's one It will be marked as a Hot Spot.
> The JIT then compiles this specific function/section into a highly optimized Native Machine Code (x86/ARM) and caches it.
> Now, if a specific machine code is unsafe, because JIT Compiler likes to do optimistic optimization compilation, and crashes during production, the app won't crash; instead, JIT Compiler will decompile the unsafe part and return to safe Interpreter mode, this is called Deoptimization.
> This is why Just-In-Time Compiled languages (Java, C#, etc) so efficient for long-running enterprise & banking software, Hot Spot compilation. They can adapt to runtime & production surprises using Interpreted mode, and can nearly reach compiled languages' speed in the long run because of JIT Compilation.

# Documentation of JVM Architecture & Magic Behind WORA - Day 2
> **Shalom**! 🐧 I have returned from the Indonesian and Civics Midterm, and tomorrow I have preparation for a Way of the Cross. So let's slap this out of the way first, so I don't think about JVM on the stations 🐧💀

## 1. Java takes a lot of weight off your shoulders... To put it on your legs 🐧
> What does that mean? In C, you manage memory manually, ```free()```, ```malloc()```, and getting user input must have a buffer size; you need to set the maximum and code the edge cases of memory yourself, powerful, but you can shoot your foot.
> In Java, the mindset is different. They take a lot of weight off your shoulders during development time, no guessing memory size, handling edge cases, Java has `try`, `catch`, `throw`, `finally`, graceful error handling, and it'll throw an exception, no Undefined Behavior ([[Undefined Behavior|Undefined Behavior]])
> BUT, it puts the weight on your legs, it's Just-In-Time Compiled, unlike C, which has Ahead-Of-Time Compilation, so the weight is put during production. Even though Java is efficient, it's not as fast as C

## 2. Java Primitives and Object
So, now remember what we studied about pointers ([[Pointer|Pointer]])? We touched Java, right? Yes, primitives are essentially the same as data types in C; you would encounter types such as `int`, `char`, `float`, and `double`, among others. And as a note, Java is an OOP language and doesn't have a Struct type.
> Primitives are the same as they're in C; they can live on the Stack, Heap, or the Method Area.
> However, for Object, basically every data type that starts with a Capital letter, including your made objects, is a stack pointer pointing to a set of data on the Heap. This is also what's called Java References

```Main.java
public class Main {
    public static void main(String[] args) {
        Integer intergerObject = 8;
        System.out.println(intergerObject); // Print the value to the stdout

        int integerPrimitive = 8;
        System.out.println(integerPrimitive); // Have no method to get size, as it's a primitive
    }
}
```

## 3. References & Pointers - Pointer to the Heap
> Yep, Java took weights off of your shoulders and put it on your legs, remember? So in Java, every function is pass-by-value, never pass by reference, except for one exception.

```Main.java
public class Main {
    public static void main(String[] args) {
        for (int i = 0; i < 4; i++){
            printInt(i); // Pass by value to printInt
        }
    }
    static void printInt(int integer){ // A function taking the value of an int
        System.out.println(integer); // print the value passed to the function
    }
}
```
> The only exception? Object. Objects are first-class citizens, remember they are a "pointer" to their respective Heap Object. So in a function, passing an Object will still pass by value, but for objects, the value will be a reference, not the value. So you can modify the object you're passing to a function directly.
> But you can't change where the reference is pointing to, only modify the value the reference points to
```Main.java
public class Main {
    public static void main(String[] args) {
        for (int i = 0; i < 4; i++){
            printStuffs("This is loop number: " + i); // Pass by reference to the printStuffs function.
        }
    }
    static void printStuffs(String text){ // A function taking the reference of a String, because String is an object (character object)
        System.out.println(text); // go the value by the reference and print them
    }
}
```

## 4. The Object Header - Java Tax
> No struct, only object. That's how it is in Java, manually setting your own getters and setters. However, there are some taxes you need to be aware of: Every object has a header (12-16 Bytes), consisting of Mark Word and Class/Klass word

### The Mark Word
> This one is deeply related to the JVM, and stores data related to and used by the JVM. This can change depending on the state and JVM behaviors.

It consists of:

* Synchronization/Locking information: basically a Mutex of the Java world for multi-threaded programs. This information stores the locking and handles synchronization for threads.
* Garbage Collection (GC) metadata: like how many sweeping GC cycles has this survived, etc
* Identity Hashcode: In Java, the memory address is useless since GC rearranges things often. But Identity Hashcode is a unique identity for each object; it shows what object this is, not where, but its identity.

### The Klass Word
> This is a pointer to the Object Metadata, basically a pointer to the information of this class (public, static, type of class, etc)

## 5. Pointer Arithmetic & Array Decaying - Nope

> Remember! No Pointer Arithmetic & Array decaying, weights off of your shoulders, on your legs.

> Pointer Arithmethic is just jump ```sizeof(data_type)``` to the next value. So accessing array (```array[i]```)is just ```*(array + i)```, derefrence the location of the array + ```sizeof(element)```.
> And in Java, memory is not static; it constantly changes. Even if you somehow get an address of a variable, we don't know when it'll get invalid... So yeah, no pointer arithmetic.
> In Java, you don't do pointer arithmetic like C or operator overloading like C++; instead JVM has a special instruction that runs on every single access to get the correct data
> But isn't that slow? Remember: JIT. Java loves optimistic compilation, and the JIT is smart. If the program proves it will never reach out of bounds, it skips the checking and goes straight to machine code in which... Pointer Arithmetic in machine code 💀
> As a closing, remember what we talked about, Pass-by-reference for objects, and an  Array is an Object, so we pass the reference to the array to the function.

```Main.java
public class Main {
    public static void main(String[] args) {
        int[] arr = new int[10]; // Create a new int object on the Heap
//      int[] arr = int[10]; Not allowed, arrays are full-blown objects, can't be created in the stack
        changeIntObjectValue(arr, 4, 8);
        for (int i = 0; i < arr.length; i++) {
            System.out.println("The loop number " + i + "'s value: " + arr[i]);
        }
    }
    static void changeIntObjectValue(int[] integerArray, int index, int value) {
        integerArray[index] = value;
    }
}
```

### 6. Error Handling
> In C you'd have to use ```perror()```, ```return```, ```exit()``` hell with no clean way to do error-handling. And if there is no error-handling, your program will only crash or exhibit undefined behavior.
> In Java, we have exceptions. If the user requests an index larger than the array's size, it will trigger `ArrayIndexOutOfBoundsException`, if you access a Null Object, `NullPointerException`
> There's Multi-processing and Multi-threading, but it's later aight 💀🐧

