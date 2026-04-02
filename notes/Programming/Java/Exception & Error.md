## Exception & Error - Day 1

> **Shalom!** 🐧This week has been very busy for me. And today is my free time before Maundy Thursday. So yeah, the title is the same as the material, cause I have no idea what to title Day 1🐧💀 So yeah, let's get started.



### 1. Throwable Family Tree - No `int` just `Object`.

> So in C, when you get a success, you return/get `0`, but in Java, errors are not `int`, they are `Object.` 

> And the Throwable Family Tree consists of two members:  `Error` and `Exception`.



#### Error - JVM goes Kaboom 🐧💀

> Yep, in Java, there is no error as in C's Segfault, Array out of bounds, etc. Error in Java means your computer (or JVM) is on fire 🐧💀

> `OutOfMemory` error, `StackOverflowError` are unrecoverable errors.

> Do not catch these. Why would you fix a car whose machine already exploded instead of buying a new car 🐧?

> 

#### Exception

> Now this isn't the JVM's fault; these are the things that happen inside the code, and there are two types of `Exception.`



#### Checked Exceptions

> This is the Exception that the compiler forces you to handle, and doesn't necessarily mean your JVM is on fire, but these are errors that are checked at compile time, forcing the programmer to handle them



```java

import java.io.BufferedReader;

import java.io.FileReader;



public class Main {

    public static void main(String[] args) {

        String root = System.getProperty("user.dir");

        System.out.println("Current root directory" + root);



        String path = root + "/src/Counter.java";

        System.out.println("File path: " + path);



        FileReader fileReader = new FileReader(path);

        BufferedReader bufferedReader = new BufferedReader(fileReader);



        for (int counter = 0; counter < 3; counter++)

            System.out.println(bufferedReader.readLine());



        fileReader.close();

    }

}



```



```bash

src/Main.java:12: error: unreported exception FileNotFoundException; must be caught or declared to be thrown

        FileReader fileReader = new FileReader(path);

                                ^

src/Main.java:16: error: unreported exception IOException; must be caught or declared to be thrown

            System.out.println(bufferedReader.readLine());

                                                      ^

src/Main.java:18: error: unreported exception IOException; must be caught or declared to be thrown

        fileReader.close();

                        ^

3 errors

```

> The reason that this throws an error is that the compiler checks the file and the content.

> If the file doesn't exist (which is expected in the real world), the file won´t even compile, and you need to surround these in try-catch-block which will be explained later.



#### Unchecked Exception

> As its name suggests, it doesn´t get checked during compile time; instead, it's mostly a runtime error.

> This is the most important part; this is not an error. An exception is Java gracefully crashing your program to avoid [[Undefined Behavior|Undefined Behavior]] or [[Undefined Behavior|Segfault]].



```java

public class Main {

    public static void main(String[] args) {

        int[] intArray = new int[]{1, 2, 3, 4};

        System.out.println(intArray[0]);

        System.out.println(intArray[1]);

        System.out.println(intArray[6]);

    }

}

```



```bash

1

2

Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: Index 6 out of bounds for length 4

	at Main.main(Main.java:6)



Process finished with exit code 1

``` 



```java

public class Main {

    public static void main(String[] args) {

        String newString = null;

        System.out.println(newString.length());

    }

}

```

 

```bash

Exception in thread "main" java.lang.NullPointerException: Cannot invoke "String.length()" because "newString" is null

	at Main.main(Main.java:4)



Process finished with exit code 1

```

> There are still more examples like `ArithmeticException`, `IllegalArgumentException`, and many more. And yes, they are that verbose 🐧💀



### 2. Try-Catch Block - Nested hell 🐧💀

> Try-Catch Block, a nice feature to catch Exceptions correct? But there is a `Catch`.... got it? Got it... ? 🐧💀

> Okay, but every Checked Exception **must be covered inside a Try-Catch Block**. Remember that it is important 🐧

> From our earlier example, it will throw an `IOException` and `FileNotFoundException` because Java will have no way, how to handle this command if the file isn't found, as Java throws an Exception Object.

> So to fix that, we have to add a try-catch block



```java

import java.io.BufferedReader;

import java.io.FileReader;



public class Main {

    public static void main(String[] args) {

        String root = System.getProperty("user.dir");

        System.out.println("Current root directory" + root);



        String path = root + "/src/Counter.java";

        System.out.println("File path: " + path);



        FileReader fileReader = null; // Put it outside so we can access it in all blocks

        BufferedReader bufferedReader = null;



        try{

            fileReader = new FileReader(path);

            bufferedReader = new BufferedReader(fileReader);



            for (int counter = 0; counter < 3; counter++) {

                System.out.println(bufferedReader.readLine());

            }

        }

        catch(Exception e){

            System.out.println("Exception occured:" + e);

        }

        finally {

            if  (bufferedReader != null) {

                bufferedReader.close(); // Now, GC handles Memory Management, BUT

            }

            if (fileReader != null) {

                fileReader.close(); // JVM doesn't handle File Descriptor Management

            }

            // So close, unless you want a File Descriptor leak

        }

    }

}

``` 



> Wait? I still got an `IOException`? Yep, we covered the dangerous part in a try, but look at the finally block, it's naked, it's uncovered. So to fix that? Yep nested try 🐧💀



```java

import java.io.BufferedReader;

import java.io.FileReader;



public class Main {

    public static void main(String[] args) {

        String root = System.getProperty("user.dir");

        System.out.println("Current root directory" + root);



        String path = root + "/src/Counter.java";

        System.out.println("File path: " + path);



        FileReader fileReader = null; // Put it outside so we can access it in all blocks

        BufferedReader bufferedReader = null;



        try{

            fileReader = new FileReader(path);

            bufferedReader = new BufferedReader(fileReader);



            for (int counter = 0; counter < 3; counter++) {

                System.out.println(bufferedReader.readLine());

            }

        }

        catch(Exception e){

            System.out.println("Exception occured:" + e);

        }

        finally {

            try{

                if  (bufferedReader != null) {

                    bufferedReader.close(); // Now, GC handles Memory Management, BUT

                }

                if (fileReader != null) {

                    fileReader.close(); // JVM doesn't handle File Descriptor Management

                }

                // So close, unless you want a File Descriptor leak    

            }

            catch(Exception e){

                System.out.println("Exception occured:" + e);

            }

        }

    }

}

``` 



Or if you are lazy, just...

```java

import java.io.BufferedReader;

import java.io.FileReader;



public class Main {

    public static void main(String[] args) {

        String root = System.getProperty("user.dir");

        System.out.println("Current root directory" + root);



        String path = root + "/src/Counter.java";

        System.out.println("File path: " + path);





        try{

            FileReader fileReader = new FileReader(path);

            BufferedReader bufferedReader = new BufferedReader(fileReader);



            for (int counter = 0; counter < 3; counter++) {

                System.out.println(bufferedReader.readLine());

            }

            bufferedReader.close(); // Now, GC handles Memory Management, BUT

            fileReader.close(); // JVM doesn't handle File Descriptor Management

            

        }

        catch(Exception e){

            System.out.println("Exception occured:" + e);

        }

    }

}

``` 



Yep, finally is optional, but it is what will run when Try finishes, or Catch catches an error. This led us to a Java feature: Try-With-Resources.



### 3. Try-With-Resources - Finally is finally redundant 🐧

> So to avoid having a finally nested try, we use something called Try-With-Resources.

> It basically initialized and closed your `fileReader` or anything that needs a Try-Catch Block



```java

import java.io.BufferedReader;

import java.io.FileReader;



public class Main {

    public static void main(String[] args) {

        String root = System.getProperty("user.dir");

        System.out.println("Current root directory" + root);



        String path = root + "/src/Counter.java";

        System.out.println("File path: " + path);





        try (BufferedReader bufferedReader = new BufferedReader(new FileReader(path))) {

            for (int counter = 0; counter < 22; counter++) {

                System.out.println(bufferedReader.readLine());

            }

        }

        catch(Exception e){

            System.out.println("Exception occured:" + e);

        }

    }

}

``` 



```bash

Current root directory/home/bernardus/projects/cli/my-java-cli

File path: /home/bernardus/projects/cli/my-java-cli/src/Counter.java

//import java.io.BufferedReader;

//import java.io.FileReader;

//

//public class Main {

//    public static void main(String[] args) {

//        String root = System.getProperty("user.dir");

//        System.out.println("Current root directory" + root);

//

//        String path = root + "/src/Counter.java";

//        System.out.println("File path: " + path);

//

//        FileReader fileReader = null;

//        BufferedReader bufferedReader = null;

//

//        try{

//            fileReader = new FileReader(path);

//            bufferedReader = new BufferedReader(fileReader);

//            for (int counter = 0; counter < 3; counter++)

//                System.out.println(bufferedReader.readLine());

//        }

//        catch(Exception e){

//            System.out.println("Exception found: " + e);



Process finished with exit code 0

```

> That is basically it, automatically closes FIle Descriptor, Stream, Socket, or any Objects that implement `java.lang.AutoCLoseable` or its sub-interface `Closeable` will automatically run the `.close()` method when Try finished or Catch finished.



### 4. Custom Exception - Creativity knows no bounds 🐧

> What did I just write 🐧💀? But anyway, before we go deeper, we need to know one Java keyword: `throw.`

> This doesn't exist in C, but let's compare it to `return.`



| Feature     | `return` (C/Java)  | `throw` (Java) |
| ----------- | -------------------------------------------- | -------------- |
| Destination | The immediate caller (1 level up)| The nearest `catch` block (Can be many levels)  |
| Requirement | Must match the function/method's return type | Must be an object that extends `Throwable`|
| Cleanup     | Must manually clean up before returning      | The JVM automatically triggers `finally` block during the jump |
| Intent      | Expected result of a calculation             | Signal of an error, the point of no return 🐧💀                |

> So if you want to basically say, stop this method's execution immediately, go to the nearest Catch block, use throw, because `Catch` with catch it... got it...? Sigh 🐧💀



---



> So now custom Exception, it depends on your needs: Unchecked or Checked.

> For Unchecked, which is a Runtime Exception, your Exception (Object) must extend `RuntimeException.`



```java

public class BookNotFoundException extends RuntimeException {

    private final String attemptedTitle;



    public BookNotFoundException(String title) {

        super("Could not find book with title: " + title);

        this.attemptedTitle = title;

    }

    public String getAttemptedTitle() {

        return attemptedTitle;

    }

}



```



> As for Checked, your Object needs to extend `Exception`, and it will be a checked Exception. This basically makes the programmer think about plan B.

> Both are used to prevent [[Undefined Behavior|Undefined Behavior]], but one is enforced at compile time, so the programmer can make a plan B (User Error)

> And the other is created so the programmer can know what happened when the program crashes (Developer Error)



```java

public class DuplicateBookException extends Exception {

    public DuplicateBookException(String title) {

        super("Library already contains a book titled: " + title);

    }

}

```

Chore: Learn how Java prevents Errors inside the JVM
Credit: Once again https://www.geeksforgeeks.org/java/ saves the day 🐧
