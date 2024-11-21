---
title: Difference between LiveData, StateFlow, SharedFlow, Flow.
tags: Android
article_header:
  type: cover
  image:
---


### LiveData - Hot Stream 
In LiveData, data emissions start as soon as an observer is attached. The data flow continues, and if the observer isn’t actively consuming it, it may miss out on some values. Here's an example:

Understanding Active and Inactive Observers in LiveData

Active Observer:
An observer is considered active when the associated LifecycleOwner (e.g., an Activity or Fragment) is in a started or resumed state.
In this state, LiveData sends updates to the observer.

Inactive Observer:
An observer is inactive when the LifecycleOwner is in the paused, stopped, or destroyed state.
LiveData stops sending updates to inactive observers to avoid unnecessary processing.

Pure hot streams (like WebSockets, RxJava hot observables, or live event streams) always emit data, regardless of whether there are active subscribers.

What Happens When an Observer Isn’t Actively Consuming?
- If the observer isn't in an active state (e.g., Fragment is stopped), any data emitted by the LiveData will not be delivered to that observer.
- Once the observer becomes active again (e.g., Fragment is resumed), it will only receive the latest value stored in the LiveData, not the values emitted while it was inactive.


Why LiveData is Still Considered a Hot Stream
Even with these deviations, LiveData is categorized as a hot stream because:

It persists the last emitted value.
It emits that value immediately to any observer that becomes active.
The "data producer" (e.g., the source behind the LiveData) does not restart or get recreated for every new observer, which is a key characteristic of hot streams.


kotlin

```kotlin
val liveData = MutableLiveData<Int>()

fun emitData() {
    // Simulate emitting data over time
    liveData.value = 1
    liveData.value = 2
    liveData.value = 3
}

// Observer attaches
liveData.observe(owner, Observer { value ->
    println("Observer received: $value")
})

// Start emitting data
emitData()
```
Explanation:

Here, as soon as emitData() is called, LiveData starts emitting values 1, 2, and 3.
If an observer attaches after some of the values have been emitted, it will only receive the latest value, not the values it missed.
Key Takeaway: LiveData behaves as a hot stream, meaning it’s always ready to emit data to any observer that attaches, regardless of whether it’s actively collecting or not.

### Flow - Cold Stream 
With Flow, data emissions don’t start until a collector actively collects it. This ensures that each collector gets the full sequence of values from the beginning. Here’s an example:

kotlin
```kotlin
fun emitData(): Flow<Int> = flow {
    // Emit values with a delay to simulate a stream of data
    emit(1)
    delay(100)
    emit(2)
    delay(100)
    emit(3)
}

fun observeData() {
    // Start collecting data from the flow
    emitData().collect { value ->
        println("Collector received: $value")
    }
}

// Invoke collection
observeData()
Explanation:

```

The emitData() function returns a Flow that emits values 1, 2, and 3.
The data emission won’t start until collect is called in observeData().
Each time you call emitData().collect {...}, it starts the data sequence from the beginning, meaning each collector receives the entire sequence of values.
Key Takeaway: Flow is a cold stream by default, meaning data won’t be emitted until a collector starts consuming it. Each collector receives a fresh, complete sequence, ensuring that no values are missed.


### LiveData: Not ideal for one-time events, as it re-emits the last value when an observer re-attaches, which can lead to repeated events.


1. LiveData Example - Not Ideal for One-Time Events
With LiveData, since it retains the last emitted value, re-attaching an observer will trigger the last known value again. This can lead to repeated events, which is usually undesirable for one-time events such as navigation or showing an error message.

Example with LiveData
Imagine a LiveData used for navigation events:

```kotlin

val navigationEvent = MutableLiveData<Boolean>()

fun navigate() {
    navigationEvent.value = true // Emit navigation event
}

// Observer attaches (e.g., in Activity or Fragment)
navigationEvent.observe(owner, Observer { shouldNavigate ->
    if (shouldNavigate == true) {
        // Navigate to another screen
        println("Navigating...")
        // Reset the event to prevent repeated navigation
        navigationEvent.value = false
    }
})

```
Problem: If the observer is re-attached (e.g., on rotation or configuration change), it will re-trigger the navigation event because LiveData will emit the last known value (true), causing unintended navigation.

2. Flow Example - Better for One-Time Events
With Flow, you can emit values on-demand, and each collection is independent, so values aren’t retained between collectors unless you’re using StateFlow or SharedFlow. You can even use SharedFlow to specifically control how values are retained and replayed, making it more suitable for one-time events.

Example with SharedFlow in Flow
A SharedFlow can be used here to emit one-time events. By default, SharedFlow does not retain any value and only emits when there is an active collector.

```kotlin
// Define a SharedFlow with no replay cache (one-time emission)
val navigationEvent = MutableSharedFlow<Unit>(replay = 0)

suspend fun navigate() {
    navigationEvent.emit(Unit) // Emit navigation event once
}

// Collector (e.g., in Activity or Fragment)
lifecycleScope.launchWhenStarted {
    navigationEvent.collect {
        // Navigate to another screen
        println("Navigating...")
    }
}

```
Explanation:

navigationEvent.emit(Unit) only emits when there’s an active collector.
SharedFlow with replay = 0 ensures that no value is retained, so each collector will only get new emissions, not the last value.


### Combining Data Streams
Using LiveData
When working with multiple LiveData sources, you often need to use MediatorLiveData or Transformations to combine them. This can become cumbersome and less flexible, especially when dealing with multiple sources.

Example with LiveData

```kotlin

class MyViewModel : ViewModel() {
    private val firstLiveData: MutableLiveData<Int> = MutableLiveData()
    private val secondLiveData: MutableLiveData<Int> = MutableLiveData()

    // Combine two LiveData sources using MediatorLiveData
    val combinedLiveData: MediatorLiveData<Int> = MediatorLiveData<Int>().apply {
        addSource(firstLiveData) { value = combineValues(firstLiveData.value, secondLiveData.value) }
        addSource(secondLiveData) { value = combineValues(firstLiveData.value, secondLiveData.value) }
    }

    private fun combineValues(first: Int?, second: Int?): Int {
        return (first ?: 0) + (second ?: 0)
    }
}

```
Limitations:

The code can become verbose and harder to maintain with many sources.
The combination logic needs to be explicitly handled with MediatorLiveData.
Using Flow
In Flow, combining streams is more straightforward and expressive, thanks to operators like combine, zip, and flatMap.

Example with Flow

```kotlin

class MyViewModel : ViewModel() {
    private val firstFlow = MutableStateFlow<Int>(0)
    private val secondFlow = MutableStateFlow<Int>(0)

    // Combine two Flows using combine operator
    val combinedFlow: StateFlow<Int> = combine(firstFlow, secondFlow) { first, second ->
        first + second
    }.stateIn(viewModelScope, SharingStarted.Lazily, 0)
}

```

Benefits:

More concise and readable code.
Rich set of operators makes it easy to manipulate and combine streams without boilerplate.
2. Example Use Cases
Use Case for LiveData
LiveData is great for scenarios where you need lifecycle-aware data that directly binds to UI components.

Example Use Case: Observing a list of items for a RecyclerView.

```kotlin

class ItemViewModel : ViewModel() {
    private val itemsLiveData = MutableLiveData<List<Item>>()

    fun fetchItems() {
        // Fetch items and post them to LiveData
        itemsLiveData.postValue(getItemsFromDataSource())
    }

    fun getItems(): LiveData<List<Item>> {
        return itemsLiveData
    }
}



// In Activity/Fragment
itemViewModel.getItems().observe(viewLifecycleOwner, Observer { items ->
    recyclerViewAdapter.submitList(items)
})


```

Advantages: Automatically handles lifecycle events, preventing memory leaks and crashes.
Use Case for Flow
Flow is suitable for more complex scenarios, particularly when you don't require lifecycle awareness, such as background data processing or combining multiple data sources.

Example Use Case: Fetching data from multiple APIs and combining the results.

```kotlin

class DataViewModel : ViewModel() {
    private val userFlow = flow { emit(fetchUserData()) }
    private val postsFlow = flow { emit(fetchUserPosts()) }

    val userWithPostsFlow: Flow<UserWithPosts> = combine(userFlow, postsFlow) { user, posts ->
        UserWithPosts(user, posts)
    }

    init {
        viewModelScope.launch {
            userWithPostsFlow.collect { userWithPosts ->
                // Update UI with combined user and posts data
            }
        }
    }
}
```
Advantages: Easier to combine data streams, apply transformations, and handle asynchronous data fetching without needing lifecycle awareness.

### StateFlow

- StateFlow is a hot observable data holder that emits the current state and emits updates to its state over time.
  
- It is designed for representing a single mutable state within an application, such as UI state or shared application state.
  
- StateFlow maintains the current state internally and emits it to collectors when requested or whenever the state changes.
  
- Unlike Flow, StateFlow is unicast, meaning it supports only a single active collector at a time. It sequentially emits values to its collectors.
  
- StateFlow is typically used in scenarios where you need to observe and react to changes in a single piece of mutable state, such as UI components observing changes in ViewModel state in Android applications.


In summary, while both Flow and StateFlow represent aspects of an application's state, Flow is typically used to represent a sequence of immutable values emitted over time, while StateFlow is specifically designed to represent a single piece of mutable state that can be observed reactively.

Certainly! Let's illustrate the difference between a `Flow` and a `StateFlow` with examples:

### Example 1: Using Flow

```kotlin
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    // Example Flow representing a sequence of numbers emitted over time
    val flow: Flow<Int> = (1..5).asFlow()

    // Collecting and printing each value emitted by the Flow
    flow.collect { println(it) }
}
```

In this example, we have a simple `Flow` representing a sequence of numbers `(1, 2, 3, 4, 5)`. Each number is emitted asynchronously over time. We collect and print each value emitted by the flow using the `collect` terminal operator.

### Example 2: Using StateFlow

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.flow.*

fun main() = runBlocking {
    // Example StateFlow representing a single piece of mutable state (counter)
    val stateFlow = MutableStateFlow(0)

    // Launching a coroutine to update the stateFlow every second
    launch {
        repeat(5) {
            delay(1000)
            stateFlow.value = stateFlow.value + 1
        }
    }

    // Collecting and printing each value emitted by the StateFlow
    stateFlow.collect { println("StateFlow value: $it") }
}
```

In this example, we have a `StateFlow` representing a single piece of mutable state, a counter that increments every second. We use a `MutableStateFlow` to create this stateFlow, initially set to `0`. Then, we launch a coroutine to update the `stateFlow` every second. Finally, we collect and print each value emitted by the `StateFlow` using the `collect` terminal operator.

In summary, while both `Flow` and `StateFlow` represent aspects of an application's state, `Flow` is typically used to represent a sequence of immutable values emitted over time, while `StateFlow` is specifically designed to represent a single piece of mutable state that can be observed reactively.


| **Use Case**                              | **Example**                          | **Why StateFlow**                                             |
|-------------------------------------------|--------------------------------------|----------------------------------------------------------------|
| **UI State Management**                   | Displaying list of items in a View   | Holds the latest data, ensuring UI always has current state    |
| **Screen Configuration**                  | Managing user settings (e.g., theme) | Persistently stores configuration, making UI reactive to changes |
| **Form Validation**                       | Enabling/disabling a submit button   | Tracks form data changes, enabling live validation             |
| **Network Loading States**                | Show loading, success, or error UI   | Maintains a single source of truth for current network state   |
| **Single ViewModel Data Source**          | Data shown in a RecyclerView         | Ensures data consistency across different components           |
| **Pagination State Management**           | Handling paginated API calls         | Stores and updates loading, error, and page states             |
| **Session State**                         | User login status                    | Keeps track of current user session across the app             |
| **Reactive Navigation**                   | Tab selection in BottomNav bar       | Updates UI components in response to navigation state changes  |





### SharedFlow
SharedFlow(hot stream) - name itself says it is shared, this flow can be shared by multiple consumers, I mean if multiple collect calls happening on the sharedflow there will be a single flow which will get shared across all the consumers unlike normal flow.

StateFlow in Kotlin is designed to emit values to its collectors sequentially and only supports a single active collector at a time. This design choice is intentional and is aligned with its purpose as a unicast flow.

Here are a few reasons why StateFlow does not support multiple subscribers:

- Unicast nature: StateFlow is designed to represent a single source of truth for a particular state within an application. It maintains and emits its current state to its collectors. Having multiple subscribers would introduce the possibility of multiple entities attempting to update the same state concurrently, leading to potential race conditions and inconsistent behavior.

- Predictable behavior: By allowing only one collector at a time, StateFlow ensures predictable and deterministic behavior. Each emission is guaranteed to be received by exactly one collector, avoiding potential conflicts or ambiguity that may arise with multiple subscribers.

- Intended use case: StateFlow is commonly used to represent UI state in applications such as Android. In such scenarios, having a single observer is often sufficient and aligns well with the unidirectional data flow architecture pattern commonly used in modern UI frameworks.


**When to use sharedFlow**

| **Use Case**                            | **Example**                           | **Why SharedFlow**                                             |
|-----------------------------------------|---------------------------------------|-----------------------------------------------------------------|
| **One-time UI Events**                  | Snackbar messages, navigation         | Ensures events are consumed only once, avoids stale events      |
| **Broadcasting UI Updates**             | Settings changes, theme updates       | Notifies multiple collectors without maintaining a single state |
| **Real-time Data Streams**              | GPS updates, WebSocket messages       | Emits data as it comes, suitable for real-time notifications    |
| **UI State Across Multiple Collectors** | Loading spinners across Fragments     | Allows multiple listeners to stay synchronized with minimal setup |
| **Continuous Streams without Retention**| Sensor data (e.g., location)          | Avoids retaining old data; broadcasts only latest values        |


**When to Avoid SharedFlow**
If you need to keep the last known state, StateFlow may be more suitable because it automatically retains and emits the latest value to any new collectors.


**Comparison: StateFlow vs SharedFlow**

| **Feature**                    | **StateFlow**                            | **SharedFlow**                           |
|--------------------------------|------------------------------------------|------------------------------------------|
| **Retains Latest Value**        | Always                                   | Only if `replay = 1` or more             |
| **Replays Multiple Values**     | No                                       | Yes, based on `replay` parameter         |
| **Initial Value Required**      | Yes                                      | No                                       |
| **Hot Stream**                  | Yes                                      | Yes                                      |
| **Lifecycle Awareness**         | No                                       | No                                       |
| **Thread-Safe**                 | Yes                                      | Yes                                      |
| **Use Case**                    | Representing a single state (state holder)| Representing events or multiple states   |



### What is backpressure handling ?
Backpressure handling is a mechanism used to manage the flow of data between producers and consumers when there's a disparity in the processing speed or capacity between them. In asynchronous programming, particularly when dealing with streams of data or reactive programming, backpressure ensures that data is processed efficiently without overwhelming the system.

Here's a more detailed explanation:

1. **Producer-Consumer Disparity**: In many asynchronous systems, you have producers that generate data and consumers that process it. However, the producers might generate data at a faster rate than consumers can process it, leading to a buildup of data. This disparity in processing speed can cause issues such as memory exhaustion, performance degradation, or even system crashes.

2. **Backpressure as a Solution**: Backpressure is a way to address this problem by allowing consumers to signal to producers when they're unable to keep up with the rate of data production. Instead of blindly accepting all data emitted by producers, consumers can control the flow of data by requesting data at their own pace. This way, producers can adjust their rate of data emission based on the capacity of consumers, preventing overload and ensuring efficient resource utilization.

3. **Handling Backpressure**: There are various strategies for handling backpressure, depending on the programming model and tools used:
   - **Buffering**: One common strategy is to buffer emitted data temporarily when consumers are unable to keep up. However, excessive buffering can lead to memory issues if the buffer size becomes too large.
   - **Dropping**: Another strategy is to drop excess data when consumers are unable to process it. This approach prioritizes recent data over older data.
   - **Throttling**: Throttling involves controlling the rate of data emission based on the processing capacity of consumers. Producers adjust their emission rate dynamically to match the rate at which consumers can process data.
   - **Flow Control**: Flow control mechanisms allow consumers to communicate their demand for data back to producers. This can be achieved through signaling mechanisms like reactive streams' backpressure signals or using dedicated flow control protocols.

4. **Implementation**: Backpressure handling mechanisms are often built into libraries and frameworks for asynchronous programming. For example, reactive programming libraries like RxJava, Kotlin Coroutines, and Project Reactor provide built-in support for handling backpressure in streams of data.

In summary, backpressure handling is a crucial aspect of asynchronous programming that ensures efficient data processing and resource utilization by allowing consumers to control the flow of data from producers. It prevents overload and system instability by dynamically adjusting the rate of data emission based on the processing capacity of consumers.


### Practical Use Cases in Android Development : 

Understanding hot and cold flows is fundamental in Android app development. Choose the appropriate flow type based on your app's requirements.
Use hot flows for real-time features like chat or live updates.
Opt for cold flows when data replayability and ensuring all users see the same content are critical, such as in news or on-demand content apps.
