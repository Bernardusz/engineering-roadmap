# Documentation of The 4 Pillars of OOP & Memory Management - Day 1

> Shalom! I should be doing an assignment right now, but Java is more exciting 🐧💀 So let's get this out of the way

## 1. Object-Oriented Programming - Paradigm

> So, before we get deeper into the "pillars," we need to learn what OOP is.
>
> Object Oriented Programming, as its name suggests, is a software development paradigm in which we structure/encapsulate data and methods (ways to modify data) into a single class.
>
> So data and methods to modify the data are bundled together inside a class, which allows for robust and structured software development
>
> OOP mainly exists in Backend development frameworks as they need a structured way to manage their data
>
> OOP has 4 pillars: Encapsulation, Abstraction, Inheritance, and Polymorphism.

## 2. Encapsulation

> Encapsulation is basically the principle of combining data and methods into one "capsule": Object. This principle is the main proponent of the OOP paradigm, which doesn't really surprise me since Java is an Object Ooriented Programming Language.
>
> It basically has 3 modes of hiding data:

### a. Public

> With unrestricted access, you can modify it directly from anywhere.

### b. Protected

> It's similar to Public; the only difference is that you can only access this if you're on the same package or subclasses of this class, even in a different package.

### c. Private

> Simple: Not allowed to edit or access, except from its own class.

```Main.java
public class Main {
    public static void main(String[] args) {
        Counter counter = new Counter(7, 8, 9); // Create a new instance of the class Counter called object
        counter.thirdValue = 80; // Valid - A public variable: accessible anywhere
        counter.secondValue = 90; // Valid because protected variable is only limited to the same package, and Counter.java + Main.java are in the same package
        // counter.firstValue = 100; Invalid - You can't directly modify a private variable

    }
}
```

```Counter.java
public class Counter { // The class - The blueprint for an object
    private  int firstValue;
    protected int secondValue;
    public int thirdValue;
    public Counter(int firstValue, int secondValue, int thirdValue) {
        this.firstValue = firstValue;
        this.secondValue = secondValue;
        this.thirdValue = thirdValue;
    }
    public int getFirstValue() {
        return firstValue;
    }
    public int getSecondValue() {
        return secondValue;
    }
    public int getThirdValue() {
        return thirdValue;
    }
}
```

> So that's where getter and setter come in, you can't modify Protected (Outside the same package) and Private variables directly. Getters and Setters are like an API call you make to the backend to do CRUD operations in the Database.

## 3. Abstraction

> This is getting closer to a classic Java course, huh 💀🐧 But jokes aside, Abstraction basically means it's a blueprint for other classes telling them what they need to have without telling them how.

```Main.java
abstract class Animal {
    // Abstract method (no implementation)
    public abstract void makeSound();

    // Concrete method (with implementation)
    public void sleep() {
        System.out.println("The animal is sleeping.");
    }
}

class Dog extends Animal {
    // Provides implementation for the abstract method
    @Override
    public void makeSound() {
        System.out.println("Bark");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myDog = new Dog(); // Cannot instantiate Animal directly, but can use an object of a concrete subclass
        myDog.makeSound(); // Output: Bark
        myDog.sleep();     // Output: The animal is sleeping.
        Dog myDog2 = new Dog(); // This is also valid
        myDog2.makeSound();
        myDog2.sleep();
    }
}
```

> Wait, why is it Animal myDog instead of `Dog myDog`? Aight, so Dog is an implementation of Animal. Doghas the method defined (abstract) but no implementation.
>
> In Java, you have what's called a V-table. So what's happening is: We instantiate an Object with `Dog` class size but with only `Animal` methods.
>
> And when I call the method, it follows the pointer (reference) to the Heap, accesses the Klass Pointer to check its metadata, and finds out it's actually a dog. Then it looks up the Virtual table ([[JVM Architecture & WORA]]) in the method area to find the implementation of makeSound().
>
> Just remember, Cat is an animal, Dog is an animal, and Cow is an animal.

## 4. Polymorphism

> So why bother with Abstraction? Simple: to help with Polymorphism.
>
> Polymorphism in the short term is just Many class has the same method but different implementations. So, for example, a cat meows, a dog barks, and a cow moos.

```Main.java
abstract class Animal {
    // Abstract method (no implementation)
    public abstract void makeSound();

    // Concrete method (with implementation)
    public void sleep() {
        System.out.println("The animal is sleeping.");
    }
}

class Dog extends Animal {
    // Provides implementation for the abstract method
    @Override
    public void makeSound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow");
    }
}

class Cow extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Moo");
    }
}

public class Main {
    public static void main(String[] args) {
        Dog myDog = new Dog(); // Cannot instantiate Animal directly, but can use an object of a concrete subclass
        Cat myCat = new Cat();
        Cow myCow = new Cow();

        Animal[] myAnimals = {myDog, myCat, myCow};

        for (Animal animal : myAnimals) {
            animal.makeSound();
        }
    }
}
```

> But there are 2 kinds of Polymorphism: Compile-time and Runtime polymorphism

### a. Compile-time Polymorphism

> As its name suggests, this happens during the compilation. Mainly achieved through method overloading, where methods with the same name have different parameters.

```Main.java
class Calculator{
    static int calculate(int a, int b){
        return a+b;
    }
    static int calculate(int a, int b, int c){
        return a+b+c;
    }
    static float calculate(float a, float b, float c, float d){
        return a+b+c+d;
    }
}

public class Main {
    public static void main(String[] args) {
        Calculator calculator = new Calculator();
        System.out.println(calculator.calculate(10, 10, 10, 10));
        System.out.println(calculator.calculate(10, 10, 10));
        System.out.println(calculator.calculate(10, 10));
    }
}
```

```bash
/usr/lib/jvm/java-21-openjdk-amd64/bin/java -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath /home/bernardus/projects/cli/my-java-cli/out/production/my-java-cli Main
40.0
30
20

Process finished with exit code 0
```

> During compile time, the compiler (the one who turns `.java` -> `.class`) will pick the correct method to use.

### b. Runtime Polymorphism

> Also known as dynamic method dispatch, it happens due to method overriding. Where the Child object gives a different implementation, it overrides the Parent's one; the methods' implementation is different with the same name.

```Main.java
abstract class Animal {
    // Abstract method (no implementation)
    public abstract void makeSound();

    // Concrete method (with implementation)
    public void sleep() {
        System.out.println("The animal is sleeping.");
    }
}

class Dog extends Animal {
    // Provides implementation for the abstract method
    @Override
    public void makeSound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow");
    }
}

class Cow extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Moo");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myAnimal = new Dog();
        myAnimal.makeSound();
        
        myAnimal = new Cat();
        myAnimal.makeSound();

        myAnimal = new Cow();
        myAnimal.makeSound();
    }
}
```

```bash
/usr/lib/jvm/java-21-openjdk-amd64/bin/java -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath /home/bernardus/projects/cli/my-java-cli/out/production/my-java-cli Main
Bark
Meow
Moo

Process finished with exit code 0
```

## 5. Inheritance

> The most heard, but the last I say 💀🐧 Basically, as its name suggests, the Children inherit everything. Not just method definition, but also its implementation. 
>
> The only difference between Inheritance and Abstraction is that: Abstraction methods can either be Abstract (No implementation) or Concrete (With implementation)
>
> Whilst Inheritance from concrete objects must have an implementation.

```Main.java
class Animal {
    // Abstract method (no implementation)
    // public void makeSound(); Invalid, you need to add the implementation

    public void makeSound(){
        System.out.println("I'm a sound");
    }

    // Concrete method (with implementation)
    public void sleep() {
        System.out.println("The animal is sleeping.");
    }
}

class Dog extends Animal {
    // Provides implementation for the abstract method
    @Override
    public void makeSound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow");
    }
}

class Cow extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Moo");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myAnimal = new Dog();
        myAnimal.makeSound();

        myAnimal = new Cat();
        myAnimal.makeSound();

        myAnimal = new Cow();
        myAnimal.makeSound();

        myAnimal.sleep();
    }
}
```

```bash
/usr/lib/jvm/java-21-openjdk-amd64/bin/java -Dfile.encoding=UTF-8 -Dsun.stdout.encoding=UTF-8 -Dsun.stderr.encoding=UTF-8 -classpath /home/bernardus/projects/cli/my-java-cli/out/production/my-java-cli Main
Bark
Meow
Moo
The animal is sleeping.

Process finished with exit code 0
```

> Polymorphism is the same between Inheritance and Abstraction: Same method name, different implementation.

Credit:
- https://www.geeksforgeeks.org/java/polymorphism-in-java/ - This helps me a lot 🐧🐧🐧
- https://dev.to/paulocappa/object-oriented-programming-oop-understand-the-4-pillars-with-clear-examples-3bci
- https://www.codepolitan.com/blog/apa-itu-encapsulation-prinsip-oop-yang-wajib-dipahami-programmer/

# Documentation of The 4 Pillars of OOP & Memory Management - Day 2
> **Shalom**! We got a holiday for a week, so let's finish this ASAP and chill 🐧I did a bit change of plan, instead of GC this week, we'll move it next week. We'll learn about types and GC, but for now, OOP Keywords

## 1. Java Philosophy
> Never forget, Java was created for enterprise systems out of the frustration with C++ freedom. So Java prevent ambigousness, resulting in verbosity, and problems caused by too much freedom.

## 2. Inheritance Madness
> Everything in OOP revolves around Inheritance. Just think of Inheritance as the verb or how we do it.

```Main.java
class Animal {
    // Abstract method (no implementation)
    // public void makeSound(); Invalid, you need to add the implementation

    public void makeSound(){
        System.out.println("I'm a sound");
    }

    // Concrete method (with implementation)
    public void sleep() {
        System.out.println("The animal is sleeping.");
    }
}

class Dog extends Animal {
    // Provides implementation for the abstract method
    @Override
    public void makeSound() {
        System.out.println("Bark");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow");
    }
}

class Cow extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Moo");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal myAnimal = new Dog();
        myAnimal.makeSound();

        myAnimal = new Cat();
        myAnimal.makeSound();

        myAnimal = new Cow();
        myAnimal.makeSound();

        myAnimal.sleep();
    }
}
```

> And in Inheritance, you have 3 types of Inheritance: Abstract Inheritance, Interface Inheritance, and Concrete Inheritance.

### Abstract Inheritance
> Don't get fooled by the name. The Abstraction we discussed is a design strategy; you don't need to know everything for it to work. Abstract Inheritance means you don't need to provide how the methods should function.

```Main.java
abstract class Shape {
    abstract String draw();
    abstract double area();
    abstract double perimeter();

}
class Rectangle extends Shape {
    private int side;
    Rectangle(int side) {
        this.side = side;
    }
    String draw() {
        return "Rectangle";
    }
    double area() {
        return (double)side*side;
    }
    double perimeter() {
        return (double) side * 4;
    }
}

class Circle extends Shape {
    private int radius;
    Circle(int radius) {
        this.radius = radius;
    }
    String draw() {
        return "Circle";
    }
    double area() {
        return Math.PI * (radius * radius);
    }
    double perimeter() {
        return 2 * Math.PI * radius;
    }
}
public class Main {
    public static void main(String[] args) {
        Shape[] arrayOfShape = new Shape[10]; // Abstraction: Everything is a shape, but different kind of shape
        for (int i = 0; i < arrayOfShape.length; i++) {
            if (i % 2 == 0) {
                arrayOfShape[i] = new Rectangle(i);
            }
            else{
                arrayOfShape[i] = new Circle(i);
            }
        }
        for (Shape shape : arrayOfShape) { // Polymorphism in action
            System.out.println("The type of shape: " + shape.draw());
            System.out.println("The area of the shape: " + shape.area());
            System.out.println("The perimeter of the shape: " + shape.perimeter());
            System.out.println("------------------------------------------------");
        }
    }
}
```
```bash
The type of shape: Rectangle
The area of the shape: 0.0
The perimeter of the shape: 0.0
------------------------------------------------
The type of shape: Circle
The area of the shape: 3.141592653589793
The perimeter of the shape: 6.283185307179586
------------------------------------------------
The type of shape: Rectangle
The area of the shape: 4.0
The perimeter of the shape: 8.0
------------------------------------------------
The type of shape: Circle
The area of the shape: 28.274333882308138
The perimeter of the shape: 18.84955592153876
------------------------------------------------
The type of shape: Rectangle
The area of the shape: 16.0
The perimeter of the shape: 16.0
------------------------------------------------
The type of shape: Circle
The area of the shape: 78.53981633974483
The perimeter of the shape: 31.41592653589793
------------------------------------------------
The type of shape: Rectangle
The area of the shape: 36.0
The perimeter of the shape: 24.0
------------------------------------------------
The type of shape: Circle
The area of the shape: 153.93804002589985
The perimeter of the shape: 43.982297150257104
------------------------------------------------
The type of shape: Rectangle
The area of the shape: 64.0
The perimeter of the shape: 32.0
------------------------------------------------
The type of shape: Circle
The area of the shape: 254.46900494077323
The perimeter of the shape: 56.548667764616276
------------------------------------------------
```

### Interface Inheritance
> So, Interface Inheritance basically declares methods that this class will have and have them inherit it. That's it. With a catch that a class can have many interfaces.
```Main.java
interface mammalMethod{
    void eat();
    void sleep();
//    void speak(){ Not allowed - Interface must specifically be abstract
//
//    }
}

class Human implements mammalMethod{
    public String name;
    public int age;
    Human(String name,int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public void eat() {
        System.out.println("I just ate rice!");
    }
    @Override
    public void sleep() {
        System.out.println("I just sleep!");
    }
}

class Monkey implements mammalMethod{
    public String name;
    public int age;
    Monkey(String name,int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public void eat() {
        System.out.println("Banana!");
    }
    @Override
    public void sleep() {
        System.out.println("Zzzzzzz");
    }
}

public class Main {
    public static void main(String[] args) {
        mammalMethod human = new Human("Harry", 18); // Instantiate a new object with the mammalMethod's methods

        mammalMethod monkey = new Monkey("George", 30); // Instantiate a new object with the mammalMethod's methods
        
        mammalMethod[] mammals = {human,monkey}; // Yep you can do polymorphism with interface
        for (mammalMethod m : mammals){ // But it's limited to the methods inside the interface
            m.eat();
            m.sleep();
        }
        
    }
}
```
> And a new table unlocked! If for Concrete and Abstract Inheritance you have a Virtual Table for looking at which method to implement, in Interface Inheritance, we have an Interface Table/Interface Method Table. So when you call `human.eat()`, JVM goes to the Object's metadata, sees `eat()` definition, and then jumps to the implementation in `Human` metadata

### Concrete Inheritance
> As its name suggests, this is concrete. You can Inherit everything, and if you don't override, the children will be 100% the same as their parents

```Main.java
class Parent{
    String job;
    String hobby;
    Parent(String job, String hobby){
        this.job = job;
        this.hobby = hobby;
    }
    void work(){
        System.out.println(job);
    }
    void hobby(){
        System.out.println(hobby);
    }
}

class Child extends Parent{
    String job;
    String hobby;
    Child(String job, String hobby){
        super(job, hobby);
    }
}

public class Main {
    public static void main(String[] args) {
        Parent[] Family = new Parent[3];
        Family[0] = new Parent("Cook", "Swim");
        Family[1] = new Child("Programmer", "Write books"); // Yes, a subclass can be put inside a parent class array
        Family[2] = new Parent("Artists", "Read books");

        for (Parent p : Family) {
            p.work();
            p.hobby();
        }

        Child[] Family2 = new Child[3];
        // Family2[0] = new Parent("Juice", "Write books"); Now this is invalid, as the parent class might not have the method children class have
    }
}
```
### When to use which?
> Use an Interface when you want to define behavior regardless of who is doing it. It’s a "can-do" relationship. It doesn't matter who, just what it can do.
> Use an Abstract Class when you have a base "type" that shares some code, but the type itself shouldn't exist on its own. It’s an "is-a" relationship. So a dog and a cat are animals, but don't speak the same way.
> Use a Concrete Class when the object is fully functional on its own. Basically, when you inherit it, you'll tweak only a bit.

### VTable and ITable
> So what are they? A virtual table is a table that is an internal part of the JVM, used to achieve Polymorphism. Remember the Klass word ([[JVM Architecture & WORA]]) ? That's right, for every class a JVM creates the metadata in the Method Area ([[JVM Architecture & WORA]]) and stores a hidden "pointer" in the Klass word. So when the JVM wants to object of the same concrete/abstract parents, they see the `Shape` class, and then run the method, the JVM looks at the class Klass word, and goes to the class definition and metadata, before finally executing the method.
> ITable operates on the same basis but with a higher performance tax. When you do `m.eat()`, the JVM goes to the Klass word of the class, it looks at the ITable and searches for the `mammalMethod` identifier, before finally jumping to the code.
> The real performance cost comes from that in VTable, the implementation is always the same, while in ITable, because a mammal might implement other interfaces like `think`, we need to search first where the implementation is.
> However, with HotSpot optimization - Inline caching, if you do it multiple times, it becomes as fast as VTable.

## 3. Static Java vs Static C
> Quite long for a section, huh? Java really shows its verbosity 🐧💀. So now in C, a static (or global) variabe, means it lives in the data segment, where it'll live on for the rest of the program.
> Java combines data and text segmentss into the Method Area, so the principle is the same: A Variable that'll live on for the rest of the program.

### Variables
> Remember Stack Variable (Primitives), A Reference Variable (A Reference to a heap object) ([[JVM Architecture & WORA]]), and now we have Static Variable
> It lives in the method area, and all instances of the object share this one variable.
> And a static method is quite the same, it belongs to the class (Blueprint), not an instance of a class (Object)
```Main.java
class testing {
    private static String variable = "Hello World!";
    public String testingGround;
    testing() {
        this.testingGround = variable; // Now each instances of testing have a "pointer"/reference to the variable var
    }

    public static void changeGlobalVariable(String newWord) { // Owned by the testing, not instances of it
        variable = newWord; // A static variable can only be changed by static method
    }

}

public class Main  {
    public static void main(String[] args) {
        testing testing1 = new testing(); // Create a new object
        testing testing2 = new testing(); // Create a new object

        System.out.println(testing1.testingGround); // Prints the static variable
        System.out.println(testing2.testingGround); // Prints the static variable

        testing1.testingGround = "Hello!"; // This doesn't change static variable, it only changes where the testingGround points to
        System.out.println(testing1.testingGround);
        System.out.println(testing2.testingGround);

        testing.changeGlobalVariable("Hello Bro!"); // This do
        System.out.println(testing2.testingGround); // This doesn't change to the new static variable
        // Because String is immutable, it only creates a new String object and points there.
        // Since the old Object is still there, testing2 still points to the old object, it's not live
        // It is a copy of the reference (pointer), not a "live link."
    }
}
```

### Static "Class."
> You can't declare a static class on top of the file, but you can inside another class; this is called a static nested class.

```Main.java
class Universe {
    private static String galaxyName = "Milky Way";
    private int howManyStars;
    Universe(int howManyStars) {
        this.howManyStars = howManyStars;
    }
    // STATIC NESTED CLASS
    public static class Star {
        public void identify() {
            // Can access static members of outer class
            System.out.println("I am a star in the " + galaxyName); // It can access static variable on the parent class
            // System.out.println("And I have " + howManyStars + " Friends!" );
            // ERROR: Cannot access instance variables of Universe
            // because there is no 'this' pointer to a Universe object.
        }
    }
}

public class Main  {
    public static void main(String[] args) {
        Universe.Star sun = new Universe.Star(); // No Universe object needed!
        sun.identify();
    }
}
```
> Why do we even need this? The main reason is Encapsulation. If they're both in a different class/file, they need a getter and setter, but by allowing a static nested class to edit the outer static variable, the inner class can edit a private variable directly

## 4.  Final Keyword - The final of today 💀🐧
> I'm tired 🐧💀 Genuinely, so let's end today's note with the `final` keywords
> Final = Final. No changing, no extending, no inheritance.
> A final variable cannot be changed like C's `const`
> A final method cannot be overwritten by Children
> A final class cannot be extends/inherited. An example is String; you can't extend String

```Main.java
class Human {
    final String name = "Human";
    int age;
    Human(String name, int age) {
        // this.name = name; Not allowed, a final variable cannot be changed. However, there's what's called Blank final. You can initiate a final String and reassign it in the constructor.
        this.age = age;
    }
    final private void Speak(){
        System.out.println("I think, therefore I am");
    }
}

class Robot extends Human {
    Robot(String name, int age) {
        super(name, age);
        // this.name = "Bello"; Not allowed
    }
//    @Override Not allowed, a final method cannot be overwritten by a child's method
//    final private void Speak(){
//
//    }
}

class Main { // Now Main cannot be extended for security reasons
    public static void main(String[] args) {

    }
}
```

```Main.java
final class Main { // Now Main cannot be extended for security reasons
    public static void main(String[] args) {
        
    }
}

// class MainClass extends Main {} Not allowed
```

# Documentation of The 4 Pillars of OOP & Memory Management - Day 3
> Wanna guess something? I lost today's entire note 💀🐧 So now here I am rewriting everything from scratch

## 1. What is an Object
> Let's get this finished fast, No BS: Class is a blueprint for an object, it contains the information for the object, and is stored in the Method Area
> Object is an instance of a class and is actually just a Struct with a header. In the Klass word (Located in the header), a pointer to the class metadata is stored. Inside the class metadata has a VTable, which has the index of where the function/method actually lives. And where do functions live? The Method Area: split into Metspace for the function still in Bytecode, and code cached for "Hot" function that has been run multiple times, compiled into native machine code.
> And just remember, in C and every language, everything is contagious, so an object being a struct and having a "pointer" to thier methods is nice 💀🐧

## 2. Object = Struct + Header
> Now, I still have the code, so not everything is missing. But I have already implemented full OOP in Java for these snippets, so just politely ignore them 💀🐧

```Main.java
  

public class Main {

    public static void main(String[] args) {

        Animal myAnimal = new Dog("max", 1, true);

        Dog myDog = new Dog("Max", 2, true);

  

        myDog.wiggleTail(); // Allowed because in Dog VTable there's a pointer to the wiggleTail method

        // myAnimal.wiggleTail(); Now allowed, because in the Animal class, there's no wiggleTail

  

        myAnimal.speak(); // This let out a bark, because even though it's an Animal, the JVM goes to this class's VTable, not the Animal's one

        myDog.speak(); //Bark as usual

    }

}

  
  

class Animal{ // A struct with Header (Mark word & Klass word)

    private String name; // The struct has these variables

    private int age;

  

    Animal(String name, int age){

        this.name = name;

        this.age = age;

    }

    public void speak(){ // Methods don't live in a struct, they live in the method area

        System.out.println("I think, therefore I am");

        // So when you call Animal.speak(), the JVM goes to the Klass word

        // Inside each Klass word lives the "pointer" to the methods stored in the Method Area

        // It accesses those addresses and starts executing at that index

    }

}

  

class Dog extends Animal{ // When you inherit an Object, it's basically a copy paste

    private boolean isTrained; // We create a new variable inside the Child class

    Dog(String name, int age, boolean isTrained){ // With the exception of the header

        super(name, age); // So an identical object still has its own identity

        this.isTrained = isTrained; // And add this to this.

    }

    @Override // Override will basically tell the JVM to change the speak method in the Method area

    public void speak(){ // And point to that new address/index

        System.out.println("Bark!");

    }

    public void wiggleTail(){ // New method just add new method to the method area

        System.out.println("Woof!"); // And add new address at the end of VTable

    }

  

    public void isSafe(){

        if (isTrained){

            System.out.println("Woof~!: Safe, it's trained");

        }      

        else{

            System.out.println("Woof!?: Be careful, it's not trained");

        }

    }

    // That's why VTable is faster than ITable, because the Index is almost always the same between all inheritance

    // Animal has breathe, Dog, Cat, and Wolf inherit them all, and since nobody changes how animals breathe, the Index is the same

    // For Interface Inheritance, you can have many interfaces being inherited by one object

    // That's why the index sometimes gets messy, but with Inline caching and Hotspot optimization, it's barely noticeable for long-running apps

}
```
> Now let's see what's happening behind the scenes 🐧
```main.c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Animal Animal; // This is here because Animal needs animal_meta_data and vice versa

// The "this" pointer is passed as the first argument automatically


typedef struct {
    void (*speak)(Animal* this, int volume);
    void (*eat)(Animal* this, int total);
} animal_meta_data; // Define the metadata in the Data segment, Method area equivalent

animal_meta_data AnimalMetaData; // Live in the Data Segment, Method Area equivalent

typedef struct Animal {
	// The Mark word stores everything related to GC
	// The Klass word stores information in the class Metadata
	animal_meta_data* classMetadata;

	char name[100];
	int age;
} Animal;

void* new_animal(char* name, int age){ // This is the constructure
	Animal* animal = malloc(sizeof(Animal)); // Remember, that every object is just Instance variable
	strcpy(animal->name, name); // A stack variable pointing to a heap object
	animal->age = age;
	animal->classMetadata = &AnimalMetaData;
	return animal;
}

void speak(Animal* this, int volume) { // Methods are global functions
    printf("%s barks at %d\n", this->name, volume); // They just pass the Animal inside
}

void eat(Animal* this, int total) {
    printf("%s eats %d foods\n", this->name, total);
}

void bootstrap_animal_class() { // The JVM constructs the Klass word at run time
	AnimalMetaData.eat = eat;
	AnimalMetaData.speak = speak;
}


int main(){
	bootstrap_animal_class(); // This is setting up the Klass word during runtime	
	Animal* Max = new_animal("Max", 2); // Instantiate a new object

	Max->classMetadata->eat(Max, 3); // when you call Max.eat() it performs
	// JVM goes to the Object Klass Word -> Goes to the class metadata -> Fetch the index of the function -> Execute it
	Max->classMetadata->speak(Max, 78);

	return 0;
}
```

> Just keep in mind, this is not full OOP, just Struct + Header mimicking Object

## 3. Inheritance - Struct inside Struct
> Okay, so Inheritance is just Struct inside Struct, I'm just gonna implement Concrete Inheritance, because Interface and Abstract are very familiar once you understand behind the scenes of Concrete Inheritance

```main.c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Animal Animal; // This is here because Animal needs animal_meta_data and vice versa

// The "this" pointer is passed as the first argument automatically


typedef struct {
    void (*speak)(Animal* this, int volume);
    void (*eat)(Animal* this, int total);
} animal_meta_data; // Define the metadata in the Data segment, Method area equivilant

animal_meta_data AnimalMetaData; // Live in the Data Segment, Method Area equivilant

typedef struct Animal {
	// The Mark word stores everything related to GC
	// The Klass word stores information to the class Metadata
	animal_meta_data* classMetadata;

	char name[100];
	int age;
} Animal;

void* new_animal(char* name, int age){ // This is the constructure
	Animal* animal = malloc(sizeof(Animal)); // Remember, that every object is just Instance variable
	strcpy(animal->name, name); // A stack variable pointing to a heap object
	animal->age = age;
	animal->classMetadata = &AnimalMetaData;
	return animal;
}

void speak(Animal* this, int volume) { // Methods are global functions
    printf("%s barks at %d\n", this->name, volume); // They just pass the Animal inside
}

void eat(Animal* this, int total) {
    printf("%s eats %d foods\n", this->name, total);
}

void bootstrap_animal_class() { // The JVM construct the Klass word at run time
	AnimalMetaData.eat = eat;
	AnimalMetaData.speak = speak;
}

// ----------------------------

typedef struct  Dog Dog;

typedef struct dog_meta_data{
    // 1. Same "Shape" as Animal (Same offsets!)
    animal_meta_data base;
    
    // 2. New methods specific to Dog
    void (*wiggleTail)(Dog* this);
	void (*isSafe)(Dog* this);
} dog_meta_data;

dog_meta_data DogMetaData; // Leave it empty at global scope

typedef struct Dog // New Dog class
{
	Animal parent; // The parent's struct is here
	bool isTrained; // New variable unique to Dog
} Dog;

Dog* new_dog(char* name, int age) {
    Dog* d = malloc(sizeof(Dog)); // Initialize the memory
    d->parent.classMetadata = (animal_meta_data*)&DogMetaData; // Point to Dog's specialized VTable, but have it 
    strcpy(d->parent.name, name); // Initialize name
    d->parent.age = age; // Initialize age
    d->isTrained = true; // Initialize isTrained
    return d;
}

void isSafe(Dog* this) {
	if (this->isTrained){
	    printf("%s (a Dog) is safe\n", this->parent.name);
	}
	else {
	    printf("%s (a Dog) is not trained, be careful\n", this->parent.name);
	}
}

void wiggleTail(Dog* this) {
    printf("%s wags tail!\n", this->parent.name);
}

void dog_speak(Animal* this, int volume) { // Methods are global functions
    printf("%s Barks: BARK!\n", this->name); // They just pass the Animal inside
}

void bootstrap_dog_class() {
	DogMetaData.base = AnimalMetaData; // Now this works!
	DogMetaData.base.speak = dog_speak;
	DogMetaData.wiggleTail = wiggleTail;
	DogMetaData.isSafe = isSafe;
}

int main(){
	bootstrap_animal_class(); // This is setting up the Klass word during runtime	
	bootstrap_dog_class(); // Before your process runs
	Animal* Max = new_animal("Max", 2); // Instantiate a new object

	Max->classMetadata->eat(Max, 3); // when you call Max.eat() it performs
	// JVM goes to the Object Klass Word -> Goes to the class metadata -> Fetch the index of the function -> Execute it
	Max->classMetadata->speak(Max, 78);

	Animal* Copper = (Animal *)new_dog("Copper", 3); // Polymorphism in C!
	Copper->classMetadata->eat(Copper, 2);
	
	Dog* Charlie = new_dog("Charlie", 1);
	dog_meta_data* dogVTable =(dog_meta_data *)Charlie->parent.classMetadata; // We need to create a new instance of vDogTable
	// Because the compiler will think you only have eat and speak methods

	dogVTable->wiggleTail(Charlie); // You go to the VTable of Dog when you call Charlie.wiggleTail()
	dogVTable->isSafe(Charlie);

	Animal* myAnimals[3] = {Max, Copper, (Animal*)Charlie}; // Create an array of Animal*, and you need to type cast Charlie
	for (int i = 0; i < 3; i++){
		myAnimals[i]->classMetadata->speak(myAnimals[i], i);
	}

	return 0;
}
```

> This is OOP Lite, completely Unsafe, and the absolute foundational core of OOP languages.
> It is unsafe because... No free 🐧💀 I forget, and boom, we have No Garbage Collector as well. Blind Casting (No RTTI) too: RTTI makes sure you can type cast safely... and sanely 💀🐧 If you type cast `Max` to `Dog`, and Max has no implementation of `wiggleTail()`, boom UB ([[Undefined Behavior|Undefined Behavior]]) 🐧💀
> It is OOP lite because No Encapsulation and No Exceptions for failures

## 4. This is hell, also `this.` 🐧💀
> Losing my notes is enough, but now I need to learn `this.` keyword 💀🐧
> As you can see in my C example, it's passing the address of the "Object" to the function, `this` keyword basically does that.
> `this.` stores the reference to the object, and when you call ```this.method()``` it goes to the Klass Word -> Class Metadata -> VTable and the function with the reference as param