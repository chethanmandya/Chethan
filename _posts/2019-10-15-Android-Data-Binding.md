---
title: Android Data Binding
tags: AndroidArchitectureComponents
article_header:
  type: cover
  image:
---

### Summary
You used the findViewById() function to get references to views. When your app has complex view hierarchies, findViewById() is expensive and slows down the app, because Android traverses the view hierarchy, starting at the root, until it finds the desired view. 

In this Example, you learn how to use data binding to eliminate the need for findViewById(). You also learn how to use data binding to access data directly from a view.


The code you wrote in previously uses the findViewById() function to obtain references to views.

Every time you use findViewById() to search for a view after the view is created or recreated, the Android system traverses the view hierarchy at runtime to find it. When your app has only a handful of views, this is not a problem. However, production apps may have dozens of views in a layout, and even with the best design, there will be nested views.

Think of a linear layout that contains a scroll view that contains a text view. For a large or deep view hierarchy, finding a view can take enough time that it can noticeably slow down the app for the user. Caching views in variables can help, but you still have to initialize a variable for each view, in each namespace. With lots of views and multiple activities, this adds up, too.

One solution is to create an object that contains a reference to each view. This object, called a Binding object, can be used by your whole app. This technique is called data binding. Once a binding object has been created for your app, you can access the views, and other data, through the binding object, without having to traverse the view hierarchy or search for the data.

### Data binding has the following benefits:

1. Code is shorter, easier to read, and easier to maintain than code that uses findByView().
2. Data and views are clearly separated. This benefit of data binding becomes increasingly important later in this course.
3. The Android system only traverses the view hierarchy once to get each view, and it happens during app startup, not at runtime when the user is interacting with the app.
4. You get type safety for accessing views. (Type safety means that the compiler validates types while compiling, and it throws an error if you try to assign the wrong type to a variable.)

    
<!--more-->

