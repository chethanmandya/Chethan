---
title: Kotlin Flow Key Operators and Transformations.
tags: Kotlin
---

In Kotlin, Flow is a type representing a stream of values that are sequentially emitted over time. Flow operators and transformations allow you to perform various operations on flows, such as filtering, transforming, combining, and handling errors. Let's go through some key operators and transformations with examples:


1. ### map :
   Applies a transformation to each value emitted by the flow.

   ```kotlin
   import kotlinx.coroutines.flow.*
   import kotlinx.coroutines.runBlocking

   fun main() = runBlocking {
       (1..5).asFlow()
           .map { it * it }
           .collect { println(it) } // prints: 1 4 9 16 25
   }
   ```

3. ## filter:
   Filters values emitted by the flow based on a given predicate.

   ```kotlin
   import kotlinx.coroutines.flow.*
   import kotlinx.coroutines.runBlocking

   fun main() = runBlocking {
       (1..10).asFlow()
           .filter { it % 2 == 0 }
           .collect { println(it) } // prints: 2 4 6 8 10
   }
   ```

4. ## transform:
   Allows more complex transformations by emitting multiple values and suspending the execution of the collector.

   ```kotlin
   import kotlinx.coroutines.flow.*
   import kotlinx.coroutines.runBlocking

   fun main() = runBlocking {
       (1..3).asFlow()
           .transform { value ->
               emit("A$value")
               emit("B$value")
           }
           .collect { println(it) } // prints: A1 B1 A2 B2 A3 B3
   }
   ```

5. ## flatMapConcat:
   Maps each value to a flow and concatenates the resulting flows.

   ```kotlin
   import kotlinx.coroutines.flow.*
   import kotlinx.coroutines.runBlocking

   fun main() = runBlocking {
       (1..3).asFlow()
           .flatMapConcat { value ->
               flow {
                   emit(value)
                   emit(value * 2)
               }
           }
           .collect { println(it) } // prints: 1 2 2 4 3 6
   }
   ```

6. ## zip:
   Combines corresponding values of multiple flows into pairs.

   ```kotlin
   import kotlinx.coroutines.flow.*
   import kotlinx.coroutines.runBlocking

   fun main() = runBlocking {
       val nums = (1..3).asFlow()
       val strs = flowOf("one", "two", "three")
       nums.zip(strs) { a, b -> "$a -> $b" }
           .collect { println(it) } // prints: 1 -> one 2 -> two 3 -> three
   }
   ```

7. ## catch:
   Handles exceptions thrown by upstream flows, allowing to emit alternative values or resume with a fallback.

   ```kotlin
   import kotlinx.coroutines.flow.*
   import kotlinx.coroutines.runBlocking
   import java.lang.Exception

   fun main() = runBlocking {
       flowOf(1, 2, 3)
           .map {
               if (it == 2) throw Exception("Exception occurred!")
               else it
           }
           .catch { e -> emit(0) } // fallback value
           .collect { println(it) } // prints: 1 0
   }
   ```


8. ## debounce:
   Delays emissions from the upstream flow until a specified period of time has passed without any new emissions. This is useful for scenarios like filtering out rapid, successive updates.

    ```kotlin
    import kotlinx.coroutines.delay
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        (1..5).asFlow()
            .onEach { delay(100) } // Emits every 100ms
            .debounce(200) // Debounce window of 200ms
            .collect { println(it) } // prints: 1 2 3 4 5 (emits only after 200ms of inactivity)
    }
    ```

9. ## scan:
   Accumulates values emitted by the flow over time, applying a function to each new value and the previously accumulated value.

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        (1..5).asFlow()
            .scan(0) { acc, value -> acc + value } // Accumulate values
            .collect { println(it) } // prints: 1 3 6 10 15
    }
    ```

10. ## distinctUntilChanged: 
   Filters out consecutive duplicate values emitted by the flow, allowing only distinct consecutive values.

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        flowOf(1, 1, 2, 2, 3, 3, 3, 4, 4, 4, 4)
            .distinctUntilChanged()
            .collect { println(it) } // prints: 1 2 3 4
    }
    ```

10. ## merge: 
   Merges multiple flows into a single flow, emitting values from all of them concurrently.

    ```kotlin
    import kotlinx.coroutines.delay
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        val flow1 = flowOf(1, 2, 3).onEach { delay(100) }
        val flow2 = flowOf(4, 5, 6).onEach { delay(200) }
        
        flowOf(flow1, flow2)
            .flattenMerge()
            .collect { println(it) } // prints: 1 4 2 5 3 6
    }
    ```

11. ## combine: 
   Combines values from multiple flows into a single flow of tuples, emitting a new tuple whenever any of the input flows emit a value.

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        val nums = (1..3).asFlow().onEach { delay(100) }
        val strs = flowOf("one", "two", "three").onEach { delay(200) }
        
        nums.combine(strs) { a, b -> "$a -> $b" }
            .collect { println(it) } // prints: 1 -> one 2 -> one 2 -> two 3 -> two 3 -> three
    }
    ```

12. ## take: 
   Limits the number of values emitted by the flow to a specified count.

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        (1..5).asFlow()
            .take(3)
            .collect { println(it) } // prints: 1 2 3
    }
    ```

13. ## takeWhile: 
   Emits values from the flow while the given predicate function returns true. Once the predicate returns false, it stops the emission.

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        (1..10).asFlow()
            .takeWhile { it <= 5 }
            .collect { println(it) } // prints: 1 2 3 4 5
    }
    ```

14. ## flatMapMerge: 
   Maps each value to a flow and merges the resulting flows concurrently.

    ```kotlin
    import kotlinx.coroutines.delay
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        (1..3).asFlow()
            .flatMapMerge { value ->
                flow {
                    emit(value)
                    delay(100)
                    emit(value * 2)
                }
            }
            .collect { println(it) } // prints: 1 2 2 4 3 6
    }
    ```

15. ## retry: 
   Re-subscribes to the upstream flow when an exception occurs, up to a specified number of times.

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    import java.io.IOException
    
    fun main() = runBlocking {
        flow {
            emit(1)
            throw IOException("Network error")
        }
        .retry(2) { cause -> cause is IOException }
        .catch { println("Caught exception: $it") } // prints: Caught exception: java.io.IOException: Network error
        .collect { println(it) } // Won't be executed due to exception
    }
    ```

16. ## distinct: 
   Filters out duplicate values emitted by the flow, allowing only distinct values.

    ```kotlin
    import kotlinx.coroutines.flow.*
    import kotlinx.coroutines.runBlocking
    
    fun main() = runBlocking {
        flowOf(1, 1, 2, 2, 3, 3, 3, 4, 4, 4, 4)
            .distinct()
            .collect { println(it) } // prints: 1 2 3 4
    }
    ```

These operators and transformations provide various functionalities for processing and manipulating data streams in Kotlin Flow, making it a powerful tool for handling asynchronous data in a concise and efficient manner.
