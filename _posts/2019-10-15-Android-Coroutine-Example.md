---
title: Android Room
tags: AndroidArchitectureComponents, Android Room
article_header:
  type: cover
  image:
---


### Summary

Let me start off with basic elements of thread, concurrency before I start with coroutine.
 
We all know that Synchronous basically means that you can only execute one thing at a time. where as Asynchronous means that you can execute multiple things at a time and you don't have to finish executing the current thing in order to move on to next one.
 
We also know that Concurrency or Parallelism are two different thing. in short, parallelism is about doing lot of things at once, so that we can speed up our execution. in java to enable Parallelism we uses concept of thread, thread pool - executor service. 

Concurrency is nothing but dealing with a lot of things at once, an ideal example to map this with real world problem is ticket reservation system, where multiple requests are supposed to be serve one at time although request are triggered simultaneously. in java concurrency can be achieved by using locks OR synchronized key. Anyway I don't dive this deep here, in short both Concurrency or Parallelism are made for Asynchronous.

To achieve Asynchronous, we create threads in a process. now what is this new thing called Coroutines, how does this varies from classic thread, is that something again thread ? what makes this difference. 
 
Before I tell you what is Coroutines all about and how does it work and other ? 
Let's assume you are creating client server interactive application, in which you need to make few network call to fetch some of the data, on receive data you have things to do like parse data and save data to locally for later usage. when providing such facility in your app, obviously you organise things to be Asynchronous, creating multiple worker threads is the only way to achieve Asynchronous. what happens is, when you trigger network call, each of those call will take some time to respond back with data, response could be immeadiate Or it may take a few hundred milliseconds OR even longer, during this interval your thread will be waiting state and it does nothing. 

So a thread can block, can be waiting state without doing anything for shorter interval of time when its live. there are many ways where thread go in blocking state. 

Consider a situation where your application create more than 20 threads only for server client interaction and all of those are waiting for a few seconds to receive network data. if these number of thread are waiting and not doing anything then that is inefficient use of CPU, because your CPU is ideal for those seconds while all the threads are waiting for network request to complete . 

we also have to consider below fact into account, 
- creation of a thread requires some time, thus the actual job does not start as soon as the request comes in. The client may notice a slight delay.
- Threads consume system resources (each thread consumes 1MB of memory in JVM, memory etc.), thus the system may run out of resources in case there is an unprecedented flow of client requests.

what's the core problem ? 
waiting threads don't allow scaling, threads are expensive, task waiting for IO blocks the thread itself. 

To overcome this problem, there are many framework created these days, one such is reactive programming (RxJava), Java fibers. it will help us using your cpu efficiently and only using limited number of threads. 


What we ideally got from these frameworks is a concept of where we have very light weight threads which do not consumes lot of memory. the way it works is , when you submitted particular task to this light weight thread, thread will be executing until it reaches any blocking operation like IO operation or network operation or database operation. On its block, light weight thread will be unmount task from the tread, instead of keeping it in wait, it simple take out task from the thread. while taking out, it will save the current state of the task. any local variables and stack of the task saved separately during this unmount. in this way lightweight thread will be available to up other task once it is blocked. task will be mounted back to thread when it is done with operation, by that time task will be not assigned to same thread which is unmounted, it may chose to any other available thread to perform. These task is called coroutines OR in java it is called java fibers
 
 Task is called Coroutines here. 
 
Coroutine are light-weight threads. A light weight thread means it doesn’t map on native thread, so it doesn’t require context switching on processor, so they are faster.

Kotlin implements stackless coroutines — it means that the coroutines don’t have own stack, so they don’t map on native thread.


### What the official website of Kotlin says
One can think of a coroutine as a light-weight thread. Like threads, coroutines can run in parallel, wait for each other and communicate. The biggest difference is that coroutines are very cheap, almost free: we can create thousands of them, and pay very little in terms of performance. True threads, on the other hand, are expensive to start and keep around. A thousand threads can be a serious challenge for a modern machine.



### Let’s see how we can use the Coroutines.

These are the functions to start the coroutine:

- launch{}
  - launch returns a Job and does not carry any resulting value
- async{}
  - async returns a Deferred - a light-weight non-blocking future that represents a promise to provide a result later.
 
 If the code inside the launch terminated with an exception, then it is treated like uncaught exceptions in a thread crashes Android applications. An uncaught exception inside the async code is stored inside the resulting Deferred and is not delivered anywhere else, it will get silently dropped unless processed.

### Coroutine dispatcher

To start any coroutine, you must provide dispatcher, dispatcher is nothing but indicating where you want to dispatch execution once task is completed. 

there are two types of dispatchers in Android . 

```kotlin
// dispatches execution into Android main thread
val uiDispatcher: CoroutineDispatcher = Dispatchers.Main

// represent a pool of shared threads as coroutine dispatcher
val bgDispatcher: CoroutineDispatcher = Dispatchers.I0
```

### Coroutine scope

To launch coroutine you need to provide its context on where it should launch, we have to two scopes to launch coroutines - CoroutineScope or use GlobalScope.

```kotlin
// GlobalScope example
class MainFragment : Fragment() {
    fun loadData() = GlobalScope.launch {  ...  }
}
// CoroutineScope example
class MainFragment : Fragment() {

    val uiScope = CoroutineScope(Dispatchers.Main)

    fun loadData() = uiScope.launch {  ...  }
}
// Fragment implements CoroutineScope example
class MainFragment : Fragment(), CoroutineScope {

    override val coroutineContext: CoroutineContext
        get() = Dispatchers.Main

    fun loadData() = launch {  ...  }
}

```

### launch + async (execute task)
The parent coroutine is launched via the launch function with the Main dispatcher.
The child coroutine is launched via the async function with the IO dispatcher.

```kotlin
val uiScope = CoroutineScope(Dispatchers.Main)
fun loadData() = uiScope.launch {
    view.showLoading() // ui thread
    val task = async(bgDispatcher) { // background thread
        // your blocking call
    }
    val result = task.await()
    view.showData(result) // ui thread
}
```

Above code can also be rewritten with withContext 

```kotlin
val uiScope = CoroutineScope(Dispatchers.Main)
fun loadData() = uiScope.launch {
    view.showLoading() // ui thread
    val result = withContext(bgDispatcher) { // background thread
        // your blocking call
    }
    view.showData(result) // ui thread
}
```

### launch + withContext (execute two tasks sequentially)

```kotlin
val uiScope = CoroutineScope(Dispatchers.Main)
fun loadData() = uiScope.launch {
    view.showLoading() // ui thread

    val result1 = withContext(bgDispatcher) { // background thread
        // your blocking call
    }

    val result2 = withContext(bgDispatcher) { // background thread
        // your blocking call
    }
    
    val result = result1 + result2
    
    view.showData(result) // ui thread
```


### launch + async + async (execute two tasks parallel)

The parent coroutine is launched via the launch function with the Main dispatcher.
The child coroutines are launched via the async function with the IO dispatcher.

```kotlin
val uiScope = CoroutineScope(Dispatchers.Main)
fun loadData() = uiScope.launch {
    view.showLoading() // ui thread

    val task1 = async(bgDispatcher) { // background thread
        // your blocking call
    }

    val task2 = async(bgDispatcher) { // background thread
        // your blocking call
    }

    val result = task1.await() + task2.await()

    view.showData(result) // ui thread
}
```

### launch a coroutine with a timeout - withTimeoutOrNull 

withTimeoutOrNull function which will return null in case of timeout.

```kotlin

val uiScope = CoroutineScope(Dispatchers.Main)
fun loadData() = uiScope.launch {
    view.showLoading() // ui thread
    val task = async(bgDispatcher) { // background thread
        // your blocking call
    }
    // suspend until task is finished or return null in 2 sec
    val result = withTimeoutOrNull(2000) { task.await() }
    view.showData(result) // ui thread
}

```


### Lifecycle aware coroutine scope
With a release of android architecture components, we can create lifecycle aware coroutine scope which will cancel itself when Activity#onDestroy event occurs.

```kotlin

class MainScope : CoroutineScope, LifecycleObserver {

    private val job = SupervisorJob()
    override val coroutineContext: CoroutineContext
        get() = job + Dispatchers.Main

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun destroy() = coroutineContext.cancelChildren()
}
// usage
class MainFragment : Fragment() {
    private val uiScope = MainScope()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        lifecycle.addObserver(mainScope)
    }

    private fun loadData() = uiScope.launch {
        val result = withContext(bgDispatcher) {
            // your blocking call
        }
    }
}

```


### Example of lifecycle aware coroutine scope for ViewModel

```kotlin

open class ScopedViewModel : ViewModel() {
    private val job = SupervisorJob()
    protected val uiScope = CoroutineScope(Dispatchers.Main + job)
     override fun onCleared() {
        super.onCleared()
        uiScope.coroutineContext.cancelChildren()
    }
}
// usage
class MyViewModel : ScopedViewModel() {

    private fun loadData() = uiScope.launch {
        val result = withContext(bgDispatcher) {
            // your blocking call
        }
    }
}
```


<!--more-->

