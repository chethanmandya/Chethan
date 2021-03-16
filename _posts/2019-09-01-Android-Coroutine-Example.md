---
title: Android Coroutine
tags: AndroidArchitectureComponents
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

```kotlin

- launch{}
  - launch returns a Job and does not carry any resulting value
  
- async{}
  - async returns a Deferred - a light-weight non-blocking future that represents a promise to provide a result later.
  
 ```
 
 If the code inside the launch terminated with an exception, then it is treated like uncaught exceptions in a thread crashes Android applications. An uncaught exception inside the async code is stored inside the resulting Deferred and is not delivered anywhere else, it will get silently dropped unless processed.

### Coroutine dispatcher

To start any coroutine, you must provide dispatcher, dispatcher is nothing but indicating where you want to dispatch execution once task is completed. 

To specify where the coroutines should run, Kotlin provides three dispatchers that you can use:


```kotlin

 - Dispatchers.Main - Use this dispatcher to run a coroutine on the main Android thread. This should be used only for interacting with the UI and performing quick work. Examples include calling suspend functions, running Android UI framework operations, and updating LiveData objects.

- Dispatchers.IO - This dispatcher is optimized to perform disk or network I/O outside of the main thread. Examples include using the Room component, reading from or writing to files, and running any network operations.

- Dispatchers.Default - This dispatcher is optimized to perform CPU-intensive work outside of the main thread. Example use cases include sorting a list and parsing JSON
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

Let's discurse how Android coroutines code in the login function works,

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {
        // Create a new coroutine to move the execution off the UI thread
        viewModelScope.launch(Dispatchers.IO) {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"
            loginRepository.makeLoginRequest(jsonBody)
        }
    }
}
```


 - ViewModelScope is a predefined CoroutineScope that is included with the ViewModel KTX extensions. Note that all coroutines must run in a scope. A CoroutineScope manages one or more related coroutines.
- launch is a function that creates a coroutine and dispatches the execution of its function body to the corresponding dispatcher.
- Dispatchers.IO indicates that this coroutine should be executed on a thread reserved for I/O operations.

The login function is executed as follows:

- The app calls the login function from the View layer on the main thread.
- launch creates a new coroutine, and the network request is made independently on a thread reserved for I/O operations.
- While the coroutine is running, the login function continues execution and returns, possibly before the network request is finished. Note that for simplicity, the network response is ignored for now.

### withContext() - 
You can dispatch threads with fine-grained control. You can execute code within withcontext block without making any callbacks. 

Please understand this, Using a dispatcher that uses a thread pool like Dispatchers.IO or Dispatchers.Default does not guarantee that the block executes on the same thread from top to bottom. In some situations, Kotlin coroutines might move execution to another thread after a suspend-and-resume. ***This means thread-local variables might not point to the same value for the entire withContext() block***.


### what does below code signifies 

```kotlin
runBlocking(Dispatchers.IO) {
  // Do IO work here
}
```

If you call runBlocking(Dispatchers.IO) from the main-thread, then the main-thread will be blocked while the coroutine finishes on the IO-dispatcher.


When CoroutineDispatcher is explicitly specified in the context, then the new coroutine runs in the context of the specified dispatcher while the current thread is blocked. If the specified dispatcher is an event loop of another runBlocking, then this invocation uses the outer event loop. 


### what should be output of below code ?

```kotlin

suspend fun main() {

    runBlocking {     // but this expression blocks the main thread
        println("I'm working in thread ${Thread.currentThread().name}")
        delay(5000L)  // ... while we delay for 5 seconds to keep JVM alive
    }

    val job = GlobalScope.launch(Dispatchers.Default) { // launch a new coroutine in background and continue
        println(" I'm working in thread ${Thread.currentThread().name}")
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main thread continues here immediately
    job.join() // wait until child coroutine completes
}

Output : 
--------
I'm working in thread main
Hello,
 I'm working in thread DefaultDispatcher-worker-1
World!

Process finished with exit code 0

```

In the above code,  RunBlocking blocks main thread until RunBlocking corountine finishes its job. RunBlocking has considered main thread as its coroutine dispatcher for its execution and it has waited for 5000 miliseconds. 
It is then lanuches new coroutine in background and that runs on default dispatcher. 
since we have used ***job.join()*** , it will wait until child coroutine completes, it doesn‘t closes VM until child coroutine completes. 


### what is the output of this :

```kotlin
suspend fun main() {


    val job = GlobalScope.launch(Dispatchers.Default) { // launch a new coroutine in background and continue
        println(" I'm working in thread ${Thread.currentThread().name}")
        delay(1000L)
        println("World!")
    }
    println("Hello,") // main thread continues here immediately
    job.join() // wait until child coroutine completes

    runBlocking {     // but this expression blocks the main thread
        println("I'm working in thread ${Thread.currentThread().name}")
        delay(5000L)  // ... while we delay for 5 seconds to keep JVM alive
    }


}

Output : 
-------------
Hello,
 I'm working in thread DefaultDispatcher-worker-1
World!
I'm working in thread DefaultDispatcher-worker-1

Process finished with exit code 0
‘‘‘

Since job.join() is called, runblocking going to execute on the same default thread rather on main thread. 

