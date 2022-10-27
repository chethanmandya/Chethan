---
title: Difference between LiveData, StateFlow, SharedFlow, Flow.
tags: AndroidJetPack
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(34, 139, 87), rgb(139, 34, 139))'
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(34, 139, 87 , .4), rgba(139, 34, 139, .4))'
   
---









### LiveData
LiveData is an lifecycle aware obervable data holder ( means it knows the lifecycle of the activity or an fragment) use it when you play with UI elements(views).

### StateFlow
StateFlow(hot stream)  does similar things like LiveData but it is made using flow by kotlin guys and  only difference compare to LiveData is its not lifecycle aware but this is also been solved using repeatOnLifecycle api's, So whatever LiveData can do StateFlow can do much better with power of flow's api.

### SharedFlow
SharedFlow(hot stream) - name itself says it is shared, this flow can be shared by multiple consumers, I mean if multiple collect calls happening on the sharedflow there will be a single flow which will get shared across all the consumers unlike normal flow.

### Flow
Flow (cold stream) - In general think of it like a stream of data flowing in a pipe with  both ends having a producer and consumer running on a coroutine.
