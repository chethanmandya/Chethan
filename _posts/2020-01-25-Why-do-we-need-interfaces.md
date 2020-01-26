---
title: why do you need interfaces ?
tags: AndroidArchitectureComponents, Android Room
article_header:
  type: cover
  image:
---


#### why do you need interfaces ? 
Interfaces has multiple uses, different developer has different prospective about interfaces, some says interfaces helps us for contract, helps you to do standardization, multiple inheritances, versioning, defining signature, abstraction, decoupling, enforces rules and so on, but what is the real root reasons of interface existance. 

Let me ask you this, what is good software architecture is ?, when do you know this project has good software architect?? 
answer is, when you do change at one place, other places shouldn't get effected too much. 

The test real software architect matters when your software goes to maintaince, by the time when you want to do some changes OR want introduce any new feature , other part of the software shouldn't get effect too much. In the sense, it shouldn't look for the changes in 10 different places. 

let me take example and explain you with more granular level. 

class Program {
    static void Main(String[] args) {
        DiscountedCustomer d = new DiscountedCustomer()
        SimpleCustomer x = newSimpleCustomer()
     }
 }
 
 

let's say your application has different types of customers like, golden customer, discount costomer . 

when we looking into customer library , we shouldn't think it and say there is gold customer, silver customer , discounted customer, we should just see it as Customer 
