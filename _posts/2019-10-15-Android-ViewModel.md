---
title: Android ViewModel
tags: AndroidArchitectureComponents
article_header:
  type: cover
  image:
---

### ViewModel
-------------------

A ViewModel holds your app's UI data while surviving configuration changes. Here's why that's actually useful. Rotating your phone is considered a configuration change. Configuration changes cause your whole activity to get torn down and then recreated. If you don't properly save and restore data from the destroyed activity, you may lose that data and end up with weird UI bugs or even crashes.

So enter the ViewModel, which, of course, survives configuration changes. Instead of storing all of your UI data in your activity,
put it in the ViewModel instead. Now, this helps with configuration changes, but it's also just general good software design.

One common pitfall when developing for Android is putting a lot of variables, logic, and data into your activities and fragments. This creates a large, unmaintainable mess of a class and violates the single responsibility principle.You can use ViewModels to easily
divide out that responsibility.The ViewModels will be responsible for holding all of the data that you're going to show in your UI.
And then the activity is only responsible for knowing how to draw that data to the screen and receiving user interactions, but not for processing them.

If your app loads and stores data,I suggest making a repository class, as described in the "Guide to App Architecture."
Make sure your ViewModel doesn't become bloated with too many responsibilities. To avoid this, you can create a presenter class or implement a more fully fledged clean architecture.

OK, so to make a ViewModel, you'll end up extending the ViewModel class. And then you put your UI-related instance variables that were previously in your activity into your ViewModel. Then in your activity's onCreate, you get the ViewModel from a framework utility class called ViewModel Provider.

Notice the ViewModel Provider takes an activity instance. This is the mechanism that allows you to rotate the device, get a technically new activity instance, but always ensure that activity instance is associated with the same ViewModel. With the ViewModel instance, you can use Getters and Setters to access UI data from your activity. If you want to modify the default constructor, which currently takes
no parameters, you can use a ViewModel Factory to create a custom constructor.

Now, this is the simplest use case of a ViewModel. But the ViewModel class is also designed to work well with LiveData and data binding.

Using all of these together, you can create a reactive UI, which is just a fancy way of saying a UI that automatically updates whenever the underlying data changes.

This assumes all of your data in your ViewModel that you plan to show on screen is wrapped in LiveData. You then should set up data binding as normal.

Here's an example XML with the data binding layout tag and the variable tag for your ViewModel.

Then in your activity or fragment, you associate the variables used in the XML with the binding. Here's an example with an activity.

There's one new line of code, setLifecycleOwner. This allows the binding to observe your LiveData objects in the ViewModel. And it's essentially the magic that lets the binding update whenever the LiveData updates and the ViewModel's onscreen.

You can now directly reference LiveData fields from your ViewModel in your XML. If you combine this with binding adapters, you can move much of the boilerplate logic out of your activity.

Note that this became available at Android Studio 3.1 and higher, so make sure you're on the correct version. To learn more, check out
the Introduction to LiveData in the docs.

 You should never pass contexts into ViewModels. This means no passing in fragments, activities, or views.
As you saw earlier,ViewModels can outlive your specific activity and fragment lifecycles.

Let's say that you store an activity in your ViewModel. When you rotate the screen, that activity is destroyed. You now have a ViewModel
holding a reference to a destroyed activity. And this is a memory leak. So if you find yourself needing application contexts, which outlive ViewModel lifecycles, use the Android ViewModel subclass instead.

This includes a reference to the application for you to use. OK, second tip.

ViewModels are meant to be used in addition to onSaveInstanceState.

ViewModels do not survive process shutdown due to resource restrictions. But onSaveInstance bundles do. ViewModels are great for storing huge amounts of data. 

onSaveInstanceState bundles, not so much. Use ViewModels to store as much UI data as possible so that that data doesn't need to be reloaded or regenerated during a configuration change.

onSaveInstanceState, on the other hand, should store the smallest amount of data needed to restore the UI state if the process is shut down by the framework.

So for example, you might store all of the user's data within the ViewModel but just store the user's database ID in onSaveInstanceState.


<!--more-->

