---
title: Difference-between-MVC-MVP-and-MVVM
tags: TeXt
article_header:
  type: cover
  image:
---

We got have one to one relationship between view and presenter, View and presenter work closely together, they need to have a reference to one another.

Relationship between presenter and corresponding view is defined in a contract interface class.

It present tight coupling between view and presenter . In case of this, presenter suppose to have reference of view , In the view, get the reference of presenter. whole bottom line is it is very tight coupling between view and presenter.

In MVP, Huge amount of interfaces for interaction between layers
Each interface covers interaction to the tinest detail, Can result in numberious method.
Code size is so excessive.
Presenter needs to check if the view is still "alive" when invokes methods.
Let's have look at on MVVM There is no one to one relationship what it means is, View has reference of viemodel but view model doesn't have reference of view. ViewModel doesn't know which view to refer. But how do they communicate - Something like called Live data- very powerful concept

View model exposes to streams of events to which the views can bind to.
Loose coupling between view and view model.
All the interfaces that the MVP pattern requires are dropped.
Code sizes is less.
Live Data solves issue if View is "Alive"
<!--more-->
