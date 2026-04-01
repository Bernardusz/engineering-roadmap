---
banner: "![[kiana ! (1).jpeg]]"
icon: 🐧
cover: "![[download (1).jpeg]]"
---
# Collections, Advanced Types & The Memory Cost

## Collections & The “Node” Truth Types

> **Shalom!** 🐧 Today is still a holiday, so let’s finish this ASAP so I can chill 🐧🐧🐧

### 1. The Great Java Type Split

> A bit of a mouthful 🐧 But before we dive deep into Collections and Types, we need to know something.  
> Everything is Java, apart from primitives, which are all Objects.  
> That includes arrays and everything other than types that start with a lowercase letter.  
> Everything has a Klass word and a Mark word, no exception.  
> This is why sometimes Java is memory-heavy; everything is an object.

### 2. Collections Interface - Core of all Objects

> Collections is a collection of **Objects** or an interface that is already pre-made to store and manipulate **Objects** easily.  
> Collections itself is based on six core interfaces (Remember, an interface only declares what methods they have, not their implementation):

| Column 1 | Column 2 |
| --- | --- |
| Interface | Functionality |
| Collection | Basically an Array that can grows and shrink in size depending on the needs. The parent of all interfaces on this list, and can hold same-type or different-type elements based on implementation. |
| List | A subset of Collection and nearly identical to Python’s list, It is stores object in order, and you can access them via index. |
| Set | Represents a collection of unique element, that means no duplicate |
| SortedSet | Extension of Set you can say, that stores unique element in sorted order, either by natural ordering or custom comparator |
| NavigableSet | OOP shows its nesting huh 🐧💀 This is extension of SortedSet that provides navigation methods to retrieve elements closest to a given element |
| Queue | The same as a queue or line in real life, Queue Interface’s Objects’ order are processed by First In First Out. And cannot be accessed by index, must in order |
| Deque | Wanna guess? Yep, Subset of Queue. Stands for Double-Ended Queue and basically a Queue with either Last-In-First-Out or First-In-First-Out that allows insertion, removal, and retrieval of elements from start or end. |


> I know this is not a part of the collection interface, but `Map` is an Interface that requires `<Key, Value>` pairs instead of dealing with a single object. Similar to a Python Dictionary or a JavaScript Object

### 3. List Implementation - ArrayList & LinkedList

> There are simply too many of these objects, and if I cover everything, I’ll just copy-paste from geeksforgeeks 💀🐧 So, I’ll pick what is most used in real-world Software Engineering.

#### ArrayList

> TLDR? A resizable array that can grow and shrink in size as elements are added or removed  
> Can have duplicates and store elements in order. Accessed using their indices/index

```Main.java
import java.util.ArrayList;  
  
public class Main{  
    public static void main(String args[]){  
        int[] myIntArray = new int[3]; // Stuck in the size of a 3 int  
        myIntArray[0] = 1; // Just remember Array is an object as well  
        myIntArray[1] = 2; // It has Klass word and Mark word  
        myIntArray[2] = 3;  
  
//        ArrayList<int> myList = new ArrayList<int>(); Not allowed, ArrayList is a list of objects, not primitives  
        ArrayList<Integer> myArrayList = new ArrayList<>(); // Create an ArrayList object containing Integer Objects  
        myArrayList.add(1); // This will be massive in memory as each Integer has a Klass word and a Mark word  
        myArrayList.add(2);  
        myArrayList.add(3);  
  
        System.out.println("myIntArray's length: " + myIntArray.length); // Have methods since they're an object  
        System.out.println("myArrayList's length: " + myArrayList.size()); // Have methods since they're an object  
    }  
}
```

> So, the implementation is an array, remember `realloc()` in C? Yep, it’s an Array that copies itself when it runs out of room.

#### LinkedList

> A doubly linked list where elements are stored as nodes containing data… Complex, huh 🐧💀? Basically chains.  
Just remember that what lives on the stack is just a reference variable. So an ArrayList, LinkedList, or any implementation of Collection stores the Reference Variable on the heap.

```
ArrayList/LinkedList -> The Data on the Heap (Reference Variables) -> Real Data on another heap address
```

> And what do I mean by chains? LinkedList stores nodes, and Nodes basically store the reference to the Heap data, and a reference to the nodes for the previous and next Nodes.  
> Basically, if you want to go to 26th String you need to:

1. Iterate from Index 1, and go down the chains using the next reference until you hit Index 26
2. Then, with the reference in the node, finally go to where the data lives.  
💀🐧

```Main.java
import java.util.LinkedList;  
  
public class Main{  
    public static void main(String args[]){  
        LinkedList<String> list = new LinkedList();  
        list.add("a");  
        list.add("b");  
        for (String s: list){  
            System.out.println(s);  
        }  
    }  
}
```

> Unless you often append data at the beginning of a list, avoid it. `ArrayList` is better.

### 4. Map Implementation - HashMap

> Before going any deeper, what is the Hash type? Basically, in an Array you iterate through them using Pointer Arithmetic (#2), which, behind the scene ArrayList uses.  
> But for Hash, it’s different, with `O(1)` lookup time because of the hash table.  
> Basically similar to Page Table (#3), it is a massive table per Hash that stores where the Object is in memory, with keys being the “pointer”.  
> So you call a key, then the JVM goes to the Hash Table and looks up the key, and inside the same row contains the address where the data lives. BOOM 💀🐧  
> Just remember the key in this type can’t be duplicated, while the value can

```Main.java
import java.util.HashMap;  
import java.util.Map;  
  
public class Main{  
    public static void main(String args[]){  
        HashMap<String, String> map = new HashMap<>(); // Creates a new HashMap  
        map.put("Bernardusz", "Programmer & Novelist"); // Add new Key-Value pairs  
        map.put("Mark", "Firefighter");  
  
        System.out.println(map.get("Bernardusz")); // Get the value of inside the Hash Table  
        System.out.println(map.get("Mark"));  
  
        for (Map.Entry<String, String> entry : map.entrySet()) { // Use the Map Interface to get the type for each entry of map  
            System.out.println(entry.getKey() + ": " + entry.getValue());  
        }  
    }  
}
```

### 5.  Set Implementation - HashSet

> Yep, only one is sufficient. HashSet is… Set Implementation + Use HashMap internally, and each element added is stored as a key with a dummy value.  
> This is Concrete Inheritance in action, with the Don’t Repeat Yourself principle. Why write code twice? Just make HashSet a consumer of HashMap.  
> Oh, and also since the Value of HashSet is the Key of HashMap, no duplicate value, just `.contains()`, and you can continue.

```Main.java
import java.util.HashSet;  
  
public class Main{  
    public static void main(String args[]){  
        HashSet<String> aSet = new HashSet<>();  
        aSet.add("Bernardusz");  
        aSet.add("Bernardus");  
        aSet.add("Bernardus"); // Will be ignored since, duplicates.  
  
        System.out.println(aSet);  
        System.out.println(aSet.contains("Bernardus"));  
  
        for (String s: aSet){  
            System.out.println(s);  
        }  
    }  
}
```

```bash
[Bernardus, Bernardusz]
true
Bernardus
Bernardusz
```

### 6. Type Erasure & Type Casting

> Remember, C trusts the programmer; Java doesn’t. They put the weight off your shoulders and put it on your knees.

#### C - Trust the Programmer

> In C, Type Casting is purely a Compile-time instruction, to tell the CPU to stop using the offset map for `Data A` and start using `Data B` for Adresses and Pointer Arithmethic.  
> The CPU doesn’t check anymore, just trust, danger, freedom, and a gun up your foot 🐧

#### Java - Don’t Trust the Programmer

> Casting is a runtime operation; when you do `Dog d = (Dog) myAnimal`, the compiler guarantees a specific bytecode instruction: `checkcast`. “Are you a Dog, or a child of Dog?”

##### Upcasting

> Remember, a child can be put inside a parent’s array? This is called Upcastng, allowing the children to be put alongside the parent’s class with their own implementation. Can be implicit using (Dog) or implicit, because the Child object contains parent’s object as we discussed in #8 

##### Downcasting

> When you upcast a `Dog` to an `Animal`, you lose the power of `wiggleTail` or `getRabiesShot.` In an Array of `Animal` you downcast back to a `Dog` to gain the superpowers back.  
> That’s it, a niche but needed feature for Type Casting an Object in Java  
> This must be explicit with the `(Dog)` operator.

##### Primitive Typecasting

> Unlike objects, where the variable is only a reference, primitives that live on the stack, when type-casted, actually change their size and bits.  
> When you change from an `int` to a `double,` your bytes are physically increased from 4 to 8.

```Main.java
import java.util.ArrayList;  
  
public class Main{  
    public static void main(String args[]){  
        ArrayList<Animal> animalArrayList = new ArrayList<>();  
        animalArrayList.add(new Animal("Max", "Max", 1));  
        animalArrayList.add(new Animal("Moo", "Moo", 2));  
        animalArrayList.add(new Animal("Bird", "Bird", 3));  
        animalArrayList.add(new Animal("Pig", "Pig", 4));  
        animalArrayList.add(new Dog("Charlie", "Charlie", 1));  
        // This is implicit Typecasting, specifically type casting  
        //Charlie lost its ability to bark or getRabiesShot  
        Dog dog = new Dog("Mark", "Mark", 5); // This is explicit one  
        animalArrayList.add((Animal) dog);  
  
        for  (Animal animal : animalArrayList){  
            System.out.println(animal); // Since Java doesn't know how to print your anima  
            // It prints the The_Class_Name + '@' + Integer.toHexString(hashCode)            // Remember in Java we don't have access to memory, but we have hashCode            // basically the identity of this object for the rest of its life  
            if (animal instanceof Dog){  
                Dog dog2 = (Dog) animal; // Dogs like Charlie and Mark become Dog again (Downcasting)  
                dog2.bark(); // The regain their methods/ability back  
                dog2.getRabiesShot();  
            }  
        }  
        //------------------------------------  
        int myNumber = 4; // integer is 4 bytes  
        double newNumber = (double) myNumber; // Now its value is 4.0, and the size is 8  
        System.out.println(myNumber);  
        System.out.println(newNumber);  
    }  
}  
  
class Animal{  
    String name;  
    String surname;  
    int age;  
    public Animal(String name, String surname, int age){  
        this.name = name;  
        this.surname = surname;  
        this.age = age;  
    }  
    public void eat(){  
        System.out.println("Nyam!");  
    }  
}  
  
class Dog extends Animal{  
    public Dog(String name, String surname, int age){  
        super(name, surname, age);  
    }  
    public void bark(){  
        System.out.println("Bark!");  
    }  
    public void getRabiesShot(){  
        System.out.println("Bark! (I am healthy!)");  
    }  
}
```

```bash
Animal@6d06d69c
Animal@7852e922
Animal@4e25154f
Animal@70dea4e
Dog@5c647e05
Bark!
Bark! (I am healthy!)
Dog@33909752
Bark!
Bark! (I am healthy!)
4
4.0
```

##### Type Erasure

> The last part of Day 1: Type Erasure. When you compile your `.java` into `.class`, all the types for `ArrayList<Type>` and `HashMap<String, Integer>` disappear. It’s like `void*` in C, where you have to cast it back up to print it.  
> Except in Java, they automatically do it for you, so no need to worry.  
> And remember on each class, there’s still a Klass word, the class is plastered there, so `instanceof` works.

Credit: https://www.geeksforgeeks.org/

I take a lot of reference and information from Geeks for geeks 🐧

## Advanced Types & The Tax - Day 2

> **Shalom!** Last day of holiday, we are still learning Java 🐧 Let’s get this done ASAP.

### 1. Primitives & Object

> In C, an `int` is just 4 bytes on the stack and has no methods ([[Memory|Memory Management & Size]]). Period.  
> In Java, we have primitives, which behave the same as types in C, and a wrapper, which is just an object.

- The Primitive: `int x = 5;` (4 bytes, no header)
- The Wrapper/Object `Integer y = 5;` (~24 bytes, lives on the heap, has a Mark and Klass word)

> When you have an `ArrayList<Integer>` and add an `int,` Java performs what’s called Autoboxing.  
> It converts your 4-byte `int` into an `Integer` object so it can fit. This ties directly to Type Erasure, because everything must be an Object.
> A reason why ArrayList must be an object is: [[Pointer|Pointer Arithmetic]]. Remember Type erasure? Java removes all Types during Compilation, so it wouldn’t know whether what we passed is Object References or Primitives.  
> And since ArrayList takes Object for Uniformity, one code for all, the memory cost is very high.

```Main.java
import java.lang.reflect.Array;  
import java.util.ArrayList;  
  
public class Main{  
    public static void main(String args[]){  
        ArrayList<Integer> MyArray = new ArrayList<>(); // Creates a new ArrayList  
        MyArray.add(1); // This will be autoboxed into Integer  
        MyArray.add(2);  
  
        for (int i : MyArray){ // Converts back to an int  
            System.out.println(i); // This is an int now  
        }  
  
        for (Integer i : MyArray){ // Internally an Integer  
            System.out.println(i.toString()); // Now they have methods  
        }  
  
        // Now remember that HashCode is an identity for an object?  
        // What happens if we pass a primitive to that?  
        System.out.println(  
                Integer.toHexString(System.identityHashCode(1))  
                // Get the HashCode of an int 1  
                // int 1 will be autoboxed to an Integer                // We get the Identity HashCode                // Converts an Integer to HexString (String)                // Print it        );  
    }  
}
```

```bash
1
2
1
2
2a139a55
```

### 2. The Wrapper Tax

> Remember every Object has: `The Data`, `Mark Word`, `Klass Word`, and `Padding` for 8 bytes CPU faster look up.  
> So let’s say I have an `int` on the stack and a Java `Integer` object, and we compare them.

| Feature | C `int` | Java `Integer` object |
| --- | --- | --- |
| Data | 4 Bytes | 4 Bytes |
| Mark Word | 0 Bytes | 8 Bytes (64 bits) \|\| 4 Bytes (32 bits) |
| Klass Word | 0 Bytes | 4 - 8 Bytes |
| Padding | 0 Bytes | 0 - 4 Bytes |
| Total | 4 Bytes | 16 - 24 Bytes |


> 4 - 6 times the size of a normal `int` for the same number. If you have millions of items inside an `ArrayList<Integer>`, you are using significantly more than a normal array in C.

### 3. The String Pool & Cache

> Remember when I say String is immutable in [[4 Pillars of OOP & Memory Truth]] and when I talk about Integer caching in [[Memory|Memory]] ? Yep, they are useful here.

#### String Pool

> String is Immutable in Java, hence it is safe to have a pool filled with Strings, and have JVM point them to the same array of chars on the heap ([[Pointer|Pointer]]).

```Main.java
  
public class Main{  
    public static void main(String args[]){  
        String myVariable = "Hello world!"; // Creates a new String on the String pool on the heap.  
        String referenceMyVariable = myVariable; // Copy paste the reference to the "Hello world!" not the object, but the reference to it  
        String anotherVariable = "Hello world!"; // Creates another reference to "Hello world!"  
  
        if (myVariable == anotherVariable && myVariable == referenceMyVariable){ // This checks whether they are the same object or not  
            System.out.println("myVariable == anotherVariable"); // Not their value  
            System.out.println("They all are the same objects!");  
        }  
  
        if (myVariable.equals(anotherVariable) && myVariable.equals(referenceMyVariable)){ // This checks whether they have the same value or not  
            System.out.println("myVariable.equals(anotherVariable)");  
            System.out.println("They all have the same value");  
        }  
          
    }  
}
```

```bash
myVariable == anotherVariable
They are all the same objects!
myVariable.equals(anotherVariable)
They all have the same value.
```

#### Caching

> While String is the only one that has an unlimited pool, other Wrappers are not so lucky.

| Wrapper | Cached Range | Why? |
| --- | --- | --- |
| Boolean | `TRUE` and `FALSE` | Only 2 possible values exists, Static constant |
| Byte | `-128` to `127` | The entire 1-byte range is pre-allocated |
| Short / Long | `-128` to `127` | Only the small values are cached |
| Character | `\u0000` to `\u007f` | The first 128 ASCII characthers (0-127) |
| Integer | `-128` to `127` | The standard range (can be tuned with JVM flags) |
| Float & Double | No Cache | Between  `1.0` and `2.0` there infinite possibilities. Math logic, caching one of those many numbers is illogical because the chance of you needing 2 x `1.1` is low. |
| String | String Pool | String uses String Pool, which has a HashMap-style pool table that grows as your programs run. |


```Main.java
  
public class Main{  
    public static void main(String args[]){  
        Integer smallNumber = 12;  
        Integer referenceSmallNumber = smallNumber;  
        Integer anotherNumber = 12;  
  
        if (smallNumber == anotherNumber && smallNumber == referenceSmallNumber){ // Check are they the same object  
            System.out.println("Your number is the same object as reference number");  
        }  
  
        if (smallNumber.equals(anotherNumber)){  
            System.out.println("Your number has the same value as reference number");  
        }  
  
        System.out.println("-------------------------------------------------");  
  
        Byte smallByte = 11;  
        Byte referenceSmallByte = smallByte;  
        Byte anotherSmallByte = 11;  
  
        if (smallByte == anotherSmallByte && smallByte == referenceSmallByte){ // Check are they the same object  
            System.out.println("Your number is the same object as reference number");  
        }  
  
        if (smallByte.equals(anotherSmallByte)){  
            System.out.println("Your number has the same value as reference number");  
        }  
  
        System.out.println("-------------------------------------------------");  
  
  
        Boolean smallBoolean = true;  
        Boolean referenceSmallBoolean = true;  
        if (smallBoolean == referenceSmallBoolean){  
            System.out.println("They are the same objects!");  
        }  
  
        System.out.println("-------------------------------------------------");  
  
        Character smallCharacter = 'a';  
        Character referenceCharacter = 'a';  
        if (smallCharacter == referenceCharacter){  
            System.out.println("They are the same objects!");  
        }  
  
        System.out.println("-------------------------------------------------");  
  
        Short smallShort = 1;  
        Short referenceShort = 1;  
        if (smallShort == referenceShort){  
            System.out.println("They are the same objects!");  
        }  
  
        System.out.println("-------------------------------------------------");  
  
        Long smallLong = 1L;  
        Long referenceLong = 1L;  
        if (smallLong == referenceLong){  
            System.out.println("They are the same objects!");  
        }  
  
        System.out.println("-------------------------------------------------");  
  
        Float smallFloat = 1.1f;  
        Float referenceFloat = 1.1f;  
        if (smallFloat == referenceFloat){  
            System.out.println("They are the same objects!");  
        }  
        else {  
            System.out.println("Your objects are different!");  
        }  
  
        System.out.println("-------------------------------------------------");  
  
        Double smallDouble = 1.1;  
        Double referenceDouble = 1.1;  
        if (smallDouble == referenceDouble){  
            System.out.println("They are the same objects!");  
        }  
        else {  
            System.out.println("Your objects are different!");  
        }  
    }  
}
```

```bash
Your number is the same object as reference number
Your number has the same value as reference number
-------------------------------------------------
Your number is the same object as reference number
Your number has the same value as reference number
-------------------------------------------------
They are the same objects!
-------------------------------------------------
They are the same objects!
-------------------------------------------------
They are the same objects!
-------------------------------------------------
They are the same objects!
-------------------------------------------------
Your objects are different!
-------------------------------------------------
Your objects are different!
```

### 4. Records vs. POJOs (The Struct Evolution)

> In C, a struct is just data. In Java, a struct was very verbose. You don’t have a `struct` in Java, so Plain Old Java Object (POJO) was in its place.

#### POJO (Plain Old Java Objects)

> Before Records, if you wanted to create a simple `Person` container, you had to write a `POJO`. To make it work like a real object, you had to manually write this:

```Main.java
public class Main {  
    public static void main(String[] args) {  
        Person person = new Person("John", 23); // Instantiate a new Person  
        System.out.println(person); // This returns the data using toString()  
        // Not the HashCode        // This is immutable, you can't change John's age to 24 or 25, he's stuck in 23        // Because there are no setters        System.out.println(person.name());  
        System.out.println(person.age());  
    }  
}  
  
class Person {  
    private String name;  
    private int age;  
    public Person(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
    public String name() {  
        return name;  
    }  
    public int age() {  
        return age;  
    }  
  
    // Optionally add setters if mutable.  
  
    @Override  
    public int hashCode() {  
        return super.hashCode();  
    }  
  
    public boolean equals(Object other) {  
        return super.equals(other);  
    }  
  
    public  String toString() {  
        return "name: " + name + ", age: " + age; // Or other methods, whatever you need  
    }  
}
```

```bash
name: John, age: 23
John
23
```

> For a `struct`, let’s say C wins in elegance and less verbosity 🐧💀.

#### The “Macro” Way: Lombok

> Because developers hated to write POJOs, almost every professional project uses a library called **Lombok**. You add an annotation like `@Data`, and at compile time, it generates all that boilerplate for you, nice 🐧

#### Records

> Okay, now in Object, you can’t directly access a private variable. You need to set up a getter, `toString(),` or otherwise, you’ll print the HashCode like my earlier example.  
> But in Java 16, a feature called `Record` was introduced. It is an immutable class that functions as a data.  
> It is the official fix, and is an equivalent of `const struct,` as Everything is `final` or immutable.  
> As for why it’s immutable, it is related to Hash Security and Thread Safety.  
> Hash Security: If an object never changes, you don’t need to worry about hashCode changing. Records are the perfect keys for a `HashMap`.
> As for Thread safety, you can pass a Record between threads without worrying about one thread changing the data while others read it, basically no [[Process & Threads|Data Race]]

```Main.java
public class Main {  
    public static void main(String[] args) {  
        Person person = new Person("John", 23);  
        System.out.println(person); // This returns the data inside the Record  
        // Not the HashCode        // This is immutable, you can't change John's age to 24 or 25, he's stuck in 23        System.out.println(person.name());  
        System.out.println(person.age());  
    }  
}  
  
record Person(String name, int age) {}
```

```bash
Person[name=John, age=23]
John
23
```

## The Janitor (GC) & Generations of Objects - Day 3
> **Shallom!** I am currently am stuck between tasks and assignments, so let's get this finished ASAP 🐧💀.

### 1. Reachability - Are you alive bro?
> In C, we have to `malloc` and `free`, and memory leak is when your stack pointer goes out of scope before you `free` the memory allocated.
> In Java, Garbage Collector constantly scans "Orphaned" objects, or in other terms, Garbage. GC Roots are the anchors of your programs, that means an Object is still reachable
> If the Garbage collector can start at a root (Stack or Static variable) and follow the references to reach an Object on the Heap, it lives
> If an Object can't be followed from all the roots, it will be marked for the janitor to clean later.

### 2. Generation - Infants die young
> So, the title may seems weird, but it's actually true. In Java there are three zones on the Heap

#### The Eden Space - Young Generation
> A place on the Heap and part of the Young generation, that is allocated for the recently created objects, a place where most infant objects die.

#### Survivor Space (S0 & S1) - Young Generation
> The place containing objects that has survived a garbage collection in Eden space are moved to one of the Survivor space

#### Tenured Generation - Old Generation
> A place where Objects that has survived multiple minor GC cycles in the young generation are promoted to the Old Generation.

### 3. Stop The World - Let me clean 🐧
> So we know there are 2 generations: Old and Young, and based on those generations, there are 2 kinds of Garbage Collection.
> However if Your PC is low on memory, the OS might tell the JVM that it's running out of RAM and the JVM might trigger a GC.
> You can also do `System.gc()` in Java, but it's not recomended. What are you trying to teach how a janitor which has been perfected over 31 years (In 1995 Java was created) 🐧💀 ?

#### Minor GC - A small slip up
> This happens when the Eden Space or the Young Generation as a whole is filled.
> When you tried to `new` and object, but Eden is full.
> The pause is usually very short (sub-millisecond to a few milliseconds)
> The Janitor quickly identifies which infants are still reachable from the Stack, moves them to a Survivor Space/Tenured Generation, and wipes the rest of Eden in one clean sweep
> Very fast, workers won't even notice the Janitor working 🐧

#### Major/Full GC - Office cleaning
> This cleans the Old/Tenured Generation or the whole heap. Usually happens when the Warehouse is full, or a large object is promoted from the Survivor Space but no room.
> This is the longest GC, Lasts from 100ms to several seconds depending on the heap size.
> Janitor has to trace every single reference across the entire memory map to ensure nothing that's still used is deleted
> This is the stutter or lag spikes you see in a Java app/game

### 4. Sandbox and Limit - Janitor has limits 🐧💀
> When you start a Java program, the JVM asks the OS for a specific chunks of memory. It won't grow infinitely, the JVM won't just `realloc` infinitely💀
> That's because it would put heavy strains on the Garbage Collector and User experience.
> There is a `-Xms` for the initial RAM the JVM grabs from the OS, and `-Xmx`, the hard celling in which the JVM will never asks more ram than this.
> This is also another reason why Java is much more heavier than Python for simple program. Java asks so many just in case, while Python only asks what's nesscecary.

 > So why don't just ask for more RAM forever? Simple, Stop-The-World. The bigger the space, the longer the GC will scan the whole space. Hence longer Stop-The-World.
 > Another reason is Predictability, Making sure GC never expands more than the limit, gives predictable Full GC time.
 > And as a matter of fun fact, the GC is lazy, it only cleans when absolutely nesscecary (Minor GC when Young Gen is full, Full GC for when Heap is full or we hit/nearly hit the hard celling)
 
 ## 📚 Library CLI - ReadMe  
 Link: https://github.com/Bernardusz/library-cli
  
### 🐧 Overview  
> This is a lightweight Command Line Interface (CLI) built to demonstrate Java's memory management and modern data structures. It transitions concepts from C (manual management) to Java (automatic management).  
  
### 🛠 Architecture  
  
1. Singleton Pattern (Library.java): Ensures a single "source of truth" for the library data. It lives on the heap for the duration of the program.  
2. Records (Book.java): Utilizes Java's record type for an immutable "const struct" equivalent. This ensures thread safety and data integrity.  
  
### Garbage Collection Logic:  
  
1. Reachability: Adding a book creates a reference from the Library instance to the Book record.  
2. Cleanup: Removing a book removes that reference. Once the reference is gone, the Janitor (GC) reclaims the memory during the next cycle (Minor or Major GC).  
  
### 🚀 How to Run  
1. `javac Main.java Library.java Book.java`  
2. `java Main`  
  
### 📖 Usage  
1. Add: Enter the full details of a book. Uses nextLine() to ensure spaces in titles and descriptions are captured.  
2. Read: Search for a book by its exact title to see its metadata.  
3. Remove: Disconnects a book from the library, marking it for Garbage Collection.  
4. Exit: Gracefully shuts down the JVM.