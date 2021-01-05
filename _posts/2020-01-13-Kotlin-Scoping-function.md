---
title: Kotlin Scoping function
tags: Android Kotlin
article_header:
  type: cover
  image:
---


#### Kotlin let, run, also, apply, with

Kotlin’s standard library includes some often-used scope functions that are so abstract that even those who have been 
programming in Kotlin for a while can have a hard time keeping them straight. In this guide, I am going to clarify four 
of these scope functions in particular


Let's understand each one of this - 


***run*** and ***let*** as transformation functions, meaning that they take the value of the object they are called against, and return a new value.

---------------------------------------------------------------------------------------------------------------------------------------------------------

### Kotlin run

- run is a transformation function, meaning that it take the value of the object it is called against, and return a new value.
- The run function exposes the value of the object that it was called from as this inside the block.

#### When you should use:
run and let as transformation functions, They take the value of the object they are called against, and return a new value.

```kotlin
var tutorial = "This is Kotlin Tutorial"
    println(tutorial) //This is Kotlin Tutorial
    tutorial = run {
        val tutorial = "This is run function"
        tutorial
    }
    println(tutorial) //This is run function
    
```
---------------------------------------------------------------------------------------------------------------------------------------------------------

### Kotlin Let 

- run is a transformation function, meaning that it take the value of the object it is called against, and return a new value.
- The let function exposes the value of the object that it was called from as `it` inside the block, while `this` is retained from the outer scope.

```kotlin
fun main(args: Array<String>) {
  var str = "Hello World" 
  str.let { 
	it + "kotlin"
        println("$it!!") //  returns the result of the lambda expression
       }
       	println(str) 
}
 
//Prints
//Hello World!!
//Hello World

```

Additionally, let is useful for checking Nullable properties as shown below.

```kotlin
var name : String? = "Kotlin let null check"
name?.let { println(it) } //prints Kotlin let null check
name = null
name?.let { println(it) } //nothing happens
```
The code inside the let expression is executed only when the property is not null. Thus let saves us from the if else null checker too!

---------------------------------------------------------------------------------------------------------------------------------------------------------


***also*** and ***apply*** are typically used when the value of the object they are called against needs to be used for some mutating operation. Any return value from the also and apply blocks is ignored, and the value of the original object is returned.

#### Kotlin also :

- it takes called object reference 
- it returns the original object instead of any modified object. Hence the return data has always the same type.

#### When you should use: 
- ***also*** and ***apply*** as mutation functions, meaning that 

```kotlin
var m = 1
m = m.also { it + 1 }.also { it + 1 }
println(m) //prints 1 
```

Example 2- 

```kotlin
data class Person(var name: String, var tutorial : String)
var person = Person("Chethan", "Kotlin")

var l = person.let { it.tutorial = "Android" }
var al = person.also { it.tutorial = "Android" }

println(l)
println(al)
println(person)

Output : 
Kotlin.unit
Person(name = Chethan, tutorial = Android)
Person(name = Chethan, tutorial = Android)
```

---------------------------------------------------------------------------------------------------------------------------------------------------------

### Kotlin apply 

- it takes called object reference 
- it returns object reference on completion.

#### When you should use: 
- Often case you use this when initializing a new object.
- ***also*** and ***apply*** as mutation functions

Example - 1 : 

```kotlin
data class Person(var name: String, var tutorial : String)
var person = Person("Chethan", "Kotlin")

person.apply { this.tutorial = "Swift" }
println(person)

Output : 
Person(name = Chethan, tutorial = Swift)
```

apply vs also : apply return modifed version of object reference , but also returns called object. 

---------------------------------------------------------------------------------------------------------------------------------------------------------

### Kotlin with  

- with is used to change instance properties without the need to call dot operator over the reference every time.

- with is used to change instance properties without the need to call dot operator over the reference every time.

```kotlin
data class Person(var name: String, var tutorial : String)
var person = Person("Chethan", "Kotlin")
with(person) {
	name = "No Name"
	tutorial = "Kotlin tutorials"
}
```

