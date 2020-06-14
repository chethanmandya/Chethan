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

### Kotlin Let 

- let takes the invoked object  as  parameter 
- returns the result of the lambda expression

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


### Kotlin run

- Similar to the let function, the run function also returns the last statement.
- Unlike let, the run function doesn’t support the it keyword.

```kotlin
var tutorial = "This is Kotlin Tutorial"
    println(tutorial) //This is Kotlin Tutorial
    tutorial = run {
        val tutorial = "This is run function"
        tutorial
    }
    println(tutorial) //This is run function
    
```

#### Kotlin also :

- it takes called object reference 
- it returns the original object instead of any modified object. Hence the return data has always the same type.

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


### Kotlin apply 

- it takes called object reference 
- it returns object reference on completion.

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

