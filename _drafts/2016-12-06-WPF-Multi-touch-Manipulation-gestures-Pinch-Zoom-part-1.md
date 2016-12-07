---
layout: post
title:  WPF Multitouch Gestures and Pinch Zoom
modified: 2016-12-07 08:00:00
categories: WPF
tags: [Manipulation, Multitouch, Touch, Gestures, Pinch, Zoom, C#, .NET]
---
The past couple days I have spent my time getting touch gestures to work on a WPF project that I am currently working on. This was surprisingly easy and difficult at the same time. This will be part 1 of a 2 part blog series which focuses on Multi-Touch in WPF applications. Part 1 will focus on simple multi touch events and part 2 will dive into a more complicated real world example with an open source project I forked on github. If you want to jump ahead go to [Part 2]({% post_url 2016-12-06-WPF-Multi-touch-Manipulation-gestures-Pinch-Zoom-part-2 %})

## Pre-Reqs ##
To easily capture Multi Touch events in WPF you will need to be using .NET 4.0 or greater. You can still capture touch events in earlier versions of .NET but it is much more difficult to implement gestures. A lot of the heavy lifting is handled in the WPF Manipulations Framework.

## Multi Touch Event Workflow ##
When you implement behaviors for a mouse you have a set of 3 events you can wire up:

- MouseDown
- MouseMove
- MouseUp

Each event can be used for different purposes and you don't need to use each event for every purpose. The same is true for Multi Touch Events. Multi Touch events are referred to Manipulations and there are several type of events but we are going to focus on a simple workflow for the purposes of this write up:

- Build Up
  - ManipulationStarted - [MSDN](https://msdn.microsoft.com/en-us/library/system.windows.uielement.manipulationstarted(v=vs.110).aspx)
- On Change
  - ManipulationDelta - [MSDN](https://msdn.microsoft.com/en-us/library/system.windows.uielement.manipulationdelta(v=vs.110).aspx)
- Clean Up or Tear Down
  - ManipulationCompleted - [MSDN](https://msdn.microsoft.com/en-us/library/system.windows.uielement.manipulationcompleted(v=vs.110).aspx)

If all of this makes sense you can skip ahead, if not we are going to dive into each event and how you will use it.

### Build Up ###
The `ManipulationStarted` and `ManipulationStarting` events are a great way to set custom flags or run any build up logic. This is more of an advanced scenario but useful. Suppose you are working on a view that needs a custom touch gesture but you don't want this gesture to affect other parts of the page. You can turn off other features on the page while we capture the gesture so you don't have any side-effects.

Adding this method is super easy and bindable for your View Models.

TODO: Just use github gist instead of syntax highlighter, it is clearly broken
XAML:
{% highlight xaml linenos %}
<xml>
</xml>
{% endhighlight %}

```<xml>
</xml>```

C#:
{% highlight c# %}
public class Foo
{
}
{% endhighlight %}
### On Change ###
TODO

### Clean Up or Tear Down ###
TODO

The best way to think of all of these events is 

Describe the workflow

1. Starting
2. Delta
3. Completed

## Configure User Control for Multi Touch ##
TODO

## Simple Translation Gesture ##
### Single Touch Translation ###
Even though we are using the multi touch events we can still use the single touch point or manipulation for simple touch gestures such as translation. Suppose you have a very large image and you want to pan across that image a single manipulation tranlsation is how you would move the viewport.

## Multi Touch Pinch and Zoom ##

### Multi Touch Multiple Manipulations ###

TODO

## Multi Touch Event Properties ##
TODO

## Build Up Events and Tear Down Events ##
TODO


We have covered the basic of touch gestures and how you can take advantage of the multi touch events via the Manipulation Events in WPF when using .NET 4.0 or greater. [Part 2]({% post_url 2016-12-06-WPF-Multi-touch-Manipulation-gestures-Pinch-Zoom-part-2 %}) of this 2 part post goes into a use case on a project I work on and how you can apply these techniques to a customer user control. Full source code available in [GitHub](http://www.github.com/) fork.

ideas:

- Talk about pitfalls that caused issues
- Capturing each multi touch point
- why we don't need to manually calculating the manipulators
- How to retrieve center point
- How to apply this to a User Control
- How this works with something a little bit more complex and how to adapt the happy path
