---
title: Communicate with other fragments
tags: Android
article_header:
  type: cover
  image:
---



#### Communicate with other fragments? 
All Fragment-to-Fragment communication is done either through a shared ViewModel or through the associated Activity. Two Fragments should never communicate directly, The recommended way to communicate between fragments is to create a shared ViewModel object. Both fragments can access the ViewModel through their containing Activity.


If you are unable to use a shared ViewModel to communicate between your Fragments you can manually implement a communication flow using interfaces. However this ends up being more work to implement and it is not easily reusable in other Fragments.

 illustrated by the following sample code

```kotlin
class SharedViewModel : ViewModel() {
    val selected = MutableLiveData<Item>()

    fun select(item: Item) {
        selected.value = item
    }
}

class MasterFragment : Fragment() {

    private lateinit var itemSelector: Selector

    // Use the 'by activityViewModels()' Kotlin property delegate
    // from the fragment-ktx artifact
    private val model: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        itemSelector.setOnClickListener { item ->
            // Update the UI
        }
    }
}

class DetailFragment : Fragment() {

    // Use the 'by activityViewModels()' Kotlin property delegate
    // from the fragment-ktx artifact
    private val model: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        model.selected.observe(viewLifecycleOwner, Observer<Item> { item ->
            // Update the UI
        })
    }
}



Notice that both fragments retrieve the activity that contains them. That way, when the fragments each get the ViewModelProvider, they receive the same SharedViewModel instance, which is scoped to this activity.

<!--more-->

