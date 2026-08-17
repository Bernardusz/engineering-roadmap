## Everything - Week 2 💀🐧
> **Shalom!** I didn't really code last week. SO now that I have a day off, I need to squeeze everything into one day 💀💀💀💀.

### 1. Extension Functions - Extend without Extend 🐧
> So remember that function itself is a [[Null Safety & Types | type]]? Now, in a class we have methods. But let's say we have a class that has the concrete method. But rather than typing the same thing multiple times we create a helper method directly on the class?

```kotlin
package io.github.bernardusz  
  
fun String.newFunction(): String{  
    return "New function $this"  
}
```

```kotlin
fun main() {  
    val name = "Kotlin".newFunction().also {  
        println(it) // "New function Kotlin"  
    }  
}
```

You don't need to create a brand new class for adding a few helper methods now 🐧🐧🐧🐧🐧.

### 2. Scope Function 
> We have talked about [[Null Safety & Types | let]]. It basically run the Object through the lambda, and return the result of the lambda with it instead of this.

#### `Apply` - Configuration
> Ever have to configure a variable that doesn't have chaining method or is too complex? Enter Apply.

It basically run your object through the lambda as `this` not `it`. So you can just

```kotlin
package io.github.bernardusz  

class Human(){  
    var name: String = ""  
    var age: Int = 0  
    fun changeName(newName: String){  
        name = newName  
    }  
    fun changeAge(newAge: Int){  
        age = newAge  
    }  
    fun printInfo(){  
        println("Name: $name, Age: $age")  
    }  
}  
  
fun main() {  
    val bob = Human().apply {  
        changeName("Bob")  
        changeAge(20)  
        printInfo()  
    }  
}
```

#### `Run` - Configure and get a result
> Basically the same as apply. But when you want to get a result immediately.

```kotlin
package io.github.bernardusz  

class Human{  
    var name: String = ""  
    var age: Int = 0  
    fun changeName(newName: String){  
        name = newName  
    }  
    fun changeAge(newAge: Int){  
        age = newAge  
    }  
    fun printInfo(){  
        println("Name: $name, Age: $age")  
    }  
    fun returnName(): String{  
        return "$name"  
    }  
}  
  
fun main() {  
    val bob = Human().run {  
        changeName("Bob")  
        changeAge(20)  
        returnName()  
    }  
    println(bob)  
}
```

#### `also` - The method chaining helper
> So you know you want to log the change in the middle of a chaining? `also` basically allows you to do side effects, usually logging using `it` and returns the original object

```kotlin
package io.github.bernardusz  

class Human{  
    var name: String = ""  
    var age: Int = 0  
    fun changeName(newName: String): Human{  
        name = newName  
        return this  
    }  
    fun changeAge(newAge: Int): Human{  
        age = newAge  
        return this  
    }  
    fun returnName(): String{  
        return "$name"  
    }  
    fun printInfo() {  
        println("Name: $name, Age: $age")  
    }  
}  
  
fun main() {  
    val bob = Human().  
            changeName("Bob").  
            changeAge(20).  
            also {  
                bob -> bob.printInfo()  
            }.  
            returnName()  
    println(bob)  
}
```

#### `with` - Calling multiple method ONCE
> So as the header implies. You most of the time calls method multiple times from the same object. Rather than it cluttering the screen, you can use `with` to group them together.

```kotlin
class Human{  
    var name: String = ""  
    var age: Int = 0  
    fun changeName(newName: String): Human{  
        name = newName  
        return this  
    }  
    fun changeAge(newAge: Int): Human{  
        age = newAge  
        return this  
    }  
    fun returnName(): String{  
        return "$name"  
    }  
    fun printInfo() {  
        println("Name: $name, Age: $age")  
    }  
}  
  
fun main() {  
    val bob = Human()  
    with(bob){  
        changeName("Bob").  
        changeAge(20).  
        also {   
bob -> bob.printInfo()  
        }  
    }}
```

> I can't find a better use case in a hurry so... yeah 💀🐧

### 3. Collection Function Programming
> So now we're entering FP methods. These are used to modify Collections without using cluttering loop. So it's just a method on an object.

#### `map` - Iterate through the Collection and return the lambda result
> So. You can filter here, but most used case is transforming a collection to something else.

```kotlin
package io.github.bernardusz  
  
import kotlin.math.pow  
  
fun main() {  
    val numbers = listOf<Int>(1, 2, 3, 4, 5)  
    val toPowerOf = numbers.map { it.toDouble().pow(2) }  
    println(toPowerOf)  
  
    val nameAndAge = mapOf("Bernardus" to 16, "Stanislaus" to 21)  
    val listOfIdentity = nameAndAge.map { ( key, value ) -> "$key is $value" }  
    println(listOfIdentity)  
}
```

#### `filter` - To... Filter 💀🐧
> SO as the name suggest, it returns the lambda result to be filtered.

```kotlin
package io.github.bernardusz  
  
fun main() {  
    val numbers = listOf<Int>(1, 2, 3, 4, 5)  
    val filtered = numbers.filter { it % 2 == 0 }  
    println(filtered)  
  
    val nameAndAge = mapOf("Bernardus" to 16, "Stanislaus" to 21)  
    val isOver18 = nameAndAge.filter { it.value > 18 }  
    println(isOver18)  
}
```

#### `flatMap` - To combine map into a single object/collection
> Flat Map basically, takes any collection, but it will always return the a flatenned list that matches your lambda.

```kotlin
package io.github.bernardusz  
  
data class User(val name: String, val hobbies: List<String>)  
  
fun main() {  
    val users = listOf(  
        User("Alice", listOf("Reading", "Hiking")),  
        User("Bob", listOf("Gaming", "Cooking"))  
    )  
  
    // Using flatMap collapses all individual list items into one single list  
    val allHobbies = users.flatMap { it.hobbies }  
    println(allHobbies)  
    // Output: [Reading, Hiking, Gaming, Cooking]  
}
```

## 4. Inline Function - Adding action to a function
> Ever know a reuseable layout in HTML and JS 💀🐧? This is it but for Kotlin function. It copies the function's bytecode directly to the call site, eliminating the memory cost of creating lambda objects on the JVM.

```kotlin
package io.github.bernardusz  
  
inline fun preparingTheServer(action: () -> Unit) {  
    println("Preparing")  
    action()  
    println("Shutting things down gracefully")  
}  
  
fun main() {  
    preparingTheServer {  
        println("Server is running")  
    }  
}
```

So I guess that's it... Time to study something else 🐧🐧🐧🐧🐧