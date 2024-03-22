---
title: Difference between LiveData, StateFlow, SharedFlow, Flow.
tags: Android
article_header:
  type: cover
  image:
---

### LiveData
LiveData is an lifecycle aware obervable data holder ( means it knows the lifecycle of the activity or an fragment) use it when you play with UI elements(views).LiveData is synchronous and operates on the main (UI) thread by default, which simplifies its usage for updating UI components directly.

LiveData is part of the Android Architecture Components and is primarily used for observing changes to data in a lifecycle-aware manner.

LiveData is synchronous and operates on the main (UI) thread by default, which simplifies its usage for updating UI components directly.

### StateFlow
StateFlow(hot stream)  does similar things like LiveData but it is made using flow by kotlin guys and  only difference compare to LiveData is its not lifecycle aware but this is also been solved using repeatOnLifecycle api's, So whatever LiveData can do StateFlow can do much better with power of flow's api.

LiveData.observe() automatically unregisters the consumer when the view goes to the STOPPED state, whereas collecting from a StateFlow or any other flow does not stop collecting automatically. To achieve the same behavior,you need to collect the flow from a Lifecycle.repeatOnLifecycle block.

StateFlow is part of the Kotlin Coroutines library and provides a flow-based API for managing and observing state changes.
It is designed to be used with Kotlin coroutines, making it suitable for asynchronous programming and working with suspending functions.

StateFlow is sometimes referred to as a "hot stream" because it emits values regardless of whether there are active subscribers or not. This contrasts with "cold streams," which only produce values when there are active subscribers.

In the case of StateFlow, when a new value is emitted, it will be delivered to all active subscribers immediately, regardless of when they subscribed. This behavior is similar to traditional event buses or subjects in reactive programming.

The term "hot stream" emphasizes that StateFlow is continuously emitting values, and subscribers can join in and receive these values at any point in time. It's like a stream of water that is flowing continuously, regardless of whether there's someone there to collect it or not.

In contrast, a "cold stream" only starts producing values when a subscriber starts listening to it. This means that if there are no subscribers, the stream remains inactive and does not emit any values.

So, when developers refer to StateFlow as a "hot stream," they are highlighting its behavior of continuously emitting values regardless of subscriber status, which can be beneficial in certain scenarios, especially when dealing with real-time or constantly changing data.

### SharedFlow
SharedFlow(hot stream) - name itself says it is shared, this flow can be shared by multiple consumers, I mean if multiple collect calls happening on the sharedflow there will be a single flow which will get shared across all the consumers unlike normal flow.


### Flow
Flow (cold stream) - In general think of it like a stream of data flowing in a pipe with  both ends having a producer and consumer running on a coroutine.


**Difference between sharedflow and stateflow**

SharedFlow and StateFlow are both provided by Kotlin coroutines as part of the kotlinx.coroutines library, and they serve as mechanisms for handling flows of data asynchronously. However, they have some differences in their behavior and usage:

1. **Mutability**:
   - **StateFlow**: StateFlow is mutable and can be updated directly by calling its `value` property. It is typically used for representing and observing state changes within a single component or module.
   - **SharedFlow**: SharedFlow is immutable and cannot be directly updated after creation. Instead, it emits values through its `emit()` function. Once created, the values emitted by a SharedFlow cannot be modified or replaced.

2. **Sharing**:
   - **StateFlow**: StateFlow is designed for sharing a single source of truth about a particular piece of data. It is often used for representing UI-related state within an Android application or for managing state within a coroutine scope.
   - **SharedFlow**: SharedFlow is designed for sharing streams of values across multiple consumers. It allows multiple subscribers to receive the same stream of data independently, and each subscriber receives its own copy of the emitted values.

3. **Cold vs. Hot**:
   - **StateFlow**: StateFlow is a hot flow, meaning it emits values regardless of whether there are active subscribers or not. It behaves like a state container that continuously emits the current state to its observers.
   - **SharedFlow**: SharedFlow is a cold flow, meaning it only starts emitting values when there are active subscribers. It behaves like a broadcast channel, where subscribers receive values from the point they subscribe.

4. **Backpressure Handling**:
   - **StateFlow**: StateFlow does not support backpressure handling directly. It emits values synchronously to its observers, and if the observer is unable to keep up with the emission rate, it may miss some updates.
   - **SharedFlow**: SharedFlow supports backpressure handling through its configuration options. It can buffer emitted values or suspend the emitter when the downstream collector is unable to keep up with the emission rate, ensuring that no data is lost.

In summary, StateFlow is primarily used for representing and observing mutable state within a single component, while SharedFlow is used for sharing streams of immutable data across multiple consumers. StateFlow is hot and continuously emits values, whereas SharedFlow is cold and only emits values when there are active subscribers. Additionally, SharedFlow provides more flexibility for handling backpressure compared to StateFlow.


**What is backpressure handling ?**
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

