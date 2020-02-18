---
title: ViewModelScope
tags: Android
article_header:
  type: cover
  image:
---



#### Why do we need View Model Scope ? 
Lets take example of it, 

      class MainFragment : Fragment() {
          fun loadData() = GlobalScope.launch { viewModel.startExecution()  }
      }

      class YourViewModel : ViewModel() {

          suspend fun startExecution() {
                  withContext(Dispatchers.IO) {
                      for (i in 0..10000000) {
                          delay(500)
                          println(i)
                      }
                  }
              }
      }
      
      
What does it do ? Function in the view model will be exeucting even if you exit from the app . because scope of this execution is bound to Global. 
      
      
Same function can be rewrite and can be keep that view model scope, now the execution of function stops as soon as you navigate out from screen, this avoid memory leak and resource leak. 


    fun startExecution() {
        viewModelScope.launch {
            withContext(Dispatchers.IO) {
                for (i in 0..10000000) {
                    delay(500)
                    println(i)
                }
            }
        }

    }




 
