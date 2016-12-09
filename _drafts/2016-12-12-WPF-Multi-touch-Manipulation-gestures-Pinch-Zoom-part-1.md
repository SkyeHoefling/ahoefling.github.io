---
layout: post
title:  WPF Multitouch Gestures and Pinch Zoom Part 1
modified: 2016-12-12 08:00:00
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

## Configure UserControl for Multi Touch (Manipulations) ##
Now that we have a basic understanding of the main manipulations events we can build the necessary hooks in our User Control to detect and handle multi touch events. The example code below creates a simple user control and has event methods that do nothing. 

In the UserControl we set the property `IsManipulationEnabled="true"` which then allows the code to properly fire the manipulation events in the UserControl partial class.

XAML:
{% highlight xml linenos %}
<UserControl x:Class="Foo.Bar"
             xmlns=”http://schemas.microsoft.com/winfx/2006/xaml/presentation”
             xmlns:x=”http://schemas.microsoft.com/winfx/2006/xaml”
             xmlns:mc=”http://schemas.openxmlformats.org/markup-compatibility/2006″
             xmlns:d=”http://schemas.microsoft.com/expression/blend/2008″
             mc:Ignorable=”d”
             d:DesignHeight="300"
             d:DesignWidth="300">
    <!-- Setting IsMAnipulationEnabled here tells the UserControl 
         to execute the manipulation methods -->
    <Rectangle IsManipulationEnabled="true" />
</UserControl>
</xml>
{% endhighlight %}

C#:
{% highlight c# linenos %}
public namespace Foo
{
    public class Bar
    {
        protected override void OnManipulationStarting(ManipulationStartingEventArgs e)
        {
            // Touch event is starting
            // Execute any special initialization code here
        }

        protected override void OnManipulationStarted(ManipulationStartedEventArgs e)
        {
            // Touch event has started
            // Execute any special initialization code here
        }
       
        protected override void OnManipulationDelta(ManipulationDeltaEventArgs e)
        {
            // Delta Code or On Change
            // - Whenever the touch points update this method is executed
            // - Execute any onChange code here
        }

        protected override void OnManipulationCompleted(ManipulationCompletedEventArgs e)
        {
            // Tear Down
            // Clean up any special events and objects that no longer need
            // to listen since our touch gesture is completed
        }
    }
}
{% endhighlight %}

Your `UserControl` is all wired up and now you can capture touch events and perform additional actions on them. 

## Manipulation EventArgs ##
Each ManipulationEvent comes with it's own set of Manipulation EventArgs and for the most part they contain the same information but the data does differ for the different cases. It is important to understand what this data means before we can start programming our Gestures. We are going to go into some common use cases if you are interested in the Manipulation EventArgs documentation you can read the MSDN.

### Single or Multi Touch Points ###
How many points are detected

### Origin or Center Point ###
The center point of all the points

## Gestures ##
### Single Touch Translation ###
Even though we are using the multi touch events we can still use the single touch point or manipulation for simple touch gestures such as translation. Suppose you have a very large image and you want to pan across that image a single manipulation tranlsation is how you would move the viewport.

### Multi Touch Multiple Manipulations ###



We have covered the basic of touch gestures and how you can take advantage of the multi touch events via the Manipulation Events in WPF when using .NET 4.0 or greater. [Part 2]({% post_url 2016-12-06-WPF-Multi-touch-Manipulation-gestures-Pinch-Zoom-part-2 %}) of this 2 part post goes into a use case on a project I work on and how you can apply these techniques to a customer user control. Full source code available in [GitHub](http://www.github.com/) fork.

ideas:

- Talk about pitfalls that caused issues
- Capturing each multi touch point
- why we don't need to manually calculating the manipulators
- How to retrieve center point
- How to apply this to a User Control
- How this works with something a little bit more complex and how to adapt the happy path
