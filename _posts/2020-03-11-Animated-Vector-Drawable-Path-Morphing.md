---
title: Animated-Vector-Drawable-Path-Morphing
tags: Android
article_header:
  type: cover
  image:
---



#### Animated-Vector-Drawable-Path-Morphing
 
 
 <div class="card">
  
  <div class="card__image">
    <img class="image" src="{{site.baseurl}}/assets/post/original_demo.gif"/>
  </div>
  
   <div class="card__image">
    <img class="image" src="{{site.baseurl}}/assets/post/demo_vedio.gif"/>
  </div>
  
</div>


 

Here’s my attempt at gathering all the required information and compressing it to something you can easily understand what is all about Animated vector drawable and how this is helping to animate some of complex animation. The reason I selected this topic is that, Google has not provided stright forward examples to build animation, You may find bits of information here and there, but nothing solid.


### Insights on Vector drawable
Let me give some insights on Vector drawable, A normal Image bitmap represent as a set of pixels in a grid. with vector graphics you represent an image by describing geometrical objects, this mean you can describe an image as a set of points, lines and curves with associated color information.

Using vector graphics you will bring down your app foot print drastically to smaller size, Before 5.0, what we developer end up doing was creating multiple versions of each image for different display resolutions. this was taking more space in APK and device space while running.

With help of vector images, you only need to create an image once, as an XML file, and it can scale beauitifully for all different DPI's for different devices. you can also use same vector image for animation.

### What makes vector drawable and animated vector drawable?
Vector drawable defines a static drawable object, while animated vector drawable can add amimations to the properties of vector drawable. They both are subclasses of drawable and both are represented by XML files.

Vector drawable has few implication and trade off, vector drawable image size should be limited to with reasonable sizes because these drawables are first drawn into bitmap then uploaded to the texture on the GPU, larger bitmaps means more memory and more time to upload.

Practically, vector drawable is recommeded for icons and buttons with reasonable sizes.

When it comes to animating vector drawable, it is recommended to keep your animation short and sweet, animating path data attributes is heavy operations.

One additional information, Starting from API 25, AnimatedVectorDrawable runs on RenderThread, (as opposed to on UI thread for earlier APIs). This means animations in AnimatedVectorDrawable can remain smooth even when there is heavy workload on the UI thread.

Note: If the UI thread is unresponsive, RenderThread may continue animating until the UI thread is capable of pushing another frame. Therefore, it is not possible to precisely coordinate a RenderThread-enabled AnimatedVectorDrawable with UI thread animations.

The AnimatedVectorDrawable class (and AnimatedVectorDrawableCompat for backward-compatibility) lets you animate the properties of a vector drawable, such as rotating it or changing the path data to morph it into a different image.

### What is Path Morphing ? 

Path morphing animation technique that allows us to seamlessly transform the shapes of two paths(geometrical points) by animating the differences in their drawing commands, The most important thing to consider when implementing a path morphing animation is whether or not the paths you want to morph are compatible.

In order to morph path A into path B the following conditions must be met:

A and B have the same number of drawing commands.
The ith drawing command in A must have the same type as the ith drawing command in B, for all i.
The ith drawing command in A must have the same number of parameters as the ith drawing command in B.
You will understand these rules when you start playing around with Shape shifter tool.

### Shape Shifter tool : 

Shape Shifter is a web-app that simplifies the creation of icon animations for Android, iOS, and the web.

This tool currently exports to standalone SVGs, SVG spritesheets, and CSS keyframe animations for the web, as well as to AnimatedVectorDrawable format for Android.

https://shapeshifter.design/

There are very few simple steps to follow in shape shifter to produce your animations,

Step 1 : Get all the shapes from your designer that you wanted animated, drawable should be either in SVG/Vector drawable.

Step 2 : Import shape1 drawable and shape 2 drawable into shapes shifter tool, each of one of these svg/vector drawable has path in it. Copy those paths separtly.

Step 3 : Now you decide from what shape to what shape the animation should play, based on that you keep copied path as path-1 as from value and path-2 as end value in Path Data option in shape shifter tool.

Step 4: As soon as you mention From Path and End Path, you would see the message in the tool saying that "Path are incompatible to process". Basically this message means that two paths are having different number of geometrical points. In order to morph one shape to another, you need paths of same number of points. Shape shifter offer us ability to fix this issue instead of adding those points from ourselves. Hit the auto fix button to correct missing points. then animation is ready to play.

Step 5: Once your done, you have an export option to export it to Vector drawable, spreesheet, Animated Vector drawable.

Remaining part of how to use exported Animated Vector drawable file in your app can be learn using this sample app uploaded here.


I would like to add bit more information in coming days on how to give controller over animated vector drawable, That's one most challenging and interesting thing to do.

Youtube tutorial - https://www.youtube.com/watch?v=2aq3ljlnQdI&feature=youtu.be https://youtu.be/P35hQOsW0xU

I made sample project you to start with and understand basic, Please go through it 

https://github.com/chethu/Animated-Vector-drawable-Path-Morphing

<!--more-->
