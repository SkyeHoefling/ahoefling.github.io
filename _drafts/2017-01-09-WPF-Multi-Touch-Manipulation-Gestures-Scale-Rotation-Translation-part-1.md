---
layout: post
title:  WPF Multitouch Gestures - Translation, Scale and Rotate
modified: 2017-01-09 08:00:00
categories: WPF
tags: [Manipulation, Multitouch, Touch, Gestures, Pinch, Zoom, C#, .NET]
share: true
comments: true
---
At the end of my last project I spent some time getting touch gestures to work on a WPF application. This was surprisingly easy and difficult at the same time. This will be part 1 of a 2 part blog series which focuses on Multi-Touch in WPF applications. Part 1 will focus on simple multi touch events and part 2 will dive into a more complicated real world example with an open source project I forked on github.

## Pre-Reqs ##
To easily capture Multi Touch events in WPF you will need to be using .NET 4.0 or greater. You can still capture touch events in earlier versions of .NET but it is much more difficult to implement gestures. A lot of the heavy lifting is handled in the WPF Manipulations Framework.

## Multi Touch Event Workflow ##
When you implement behaviors for a mouse you have a set of 3 main events you can wire up:

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
    <!-- Setting IsManipulationEnabled here tells the UserControl 
         to execute the manipulation methods -->
    <Rectangle Name="BasicRect"
               Width="300"
               Height="300"
               Fill="Red"
               IsManipulationEnabled="true" />
</UserControl>
</xml>
{% endhighlight %}

C#:
{% highlight c# linenos %}
using Sytem.Windows;

public namespace Foo
{
    public partial class Bar : UserControl
    {
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

### UserControl Transform Objects ###
Before we jump into editing our Manipulation Events, we need to make sure we have the proper Transform objects set up. We want to be able to Translate, Scale and Rotate our UserControl with touch gestures so in the constructor initialize the following code.

{% highlight C# linenos %}
using System.Windows;

public namespace Foo
{
    public partial class Bar : UserControl
    {
	private TransformGroup transformGroup;
        private TranslateTransform translation;
	private ScaleTransform scale;
        private RotationTransform rotation;

        public Bar()
        {
            InitializeComponent();

            translation = new TranslationTransform(0, 0);
            scale = new ScaleTransform(1, 1);
            rotation = new RotationTransform(0);

            transformGroup.Children.Add(translation);
            transformGroup.Children.Add(scale);
            transformGroup.Children.Add(rotation);

            BasicRect.RenderTransform = transformGroup;
        }
    }
}
{% endhighlight %}

### Wiring up UserControl to Touch Events ###
The final configuration we need to perform with each touch event is notifying the `ManipulationContainer` what `UserControl` to perform the transform on. In our example here we want to maniuplate the entire `UserControl`. You will edit the `OnManipulationStarting` event with the code below

{% highlight c# linenos %}
protected override void OnManipulationStarting(ManipulationStartingEventArgs e)
{
    // Set our container to this UserControl which will allow
    // us to apply the transforms to the UserControl in the other 
    // Manipulation Events
    e.ManipulationContainer = this;
}
{% endhighlight %}

You are now ready to start applying your transforms with your Manipulaiton Events!
## Gestures ##

### Single Touch Translation ###
Even though we are using the multi touch events we can still use the single touch point or manipulation for simple touch gestures such as translation. In our example here we are defining our manipulation events inside of a `UserControl` we can simply move our `UserControl` across the screen via a single touch event's translation.

{% highlight c# linenos %}
protected override void OnManipulationDelta(ManipulationDeltaEventArgs e)
{
    // e.DeltaManipulation stores how much the touch input has
    // moved and how it has moved since the last ManipulateDelta.
    // This takes care of all the heavy lifting so we can focus on
    // just the Gesture itself
    translation.X = e.DeltaManipulation.Translation.X;
    translation.Y = e.DeltaManipulation.Translation.Y;
}
{% endhighlight %}

### Multi Touch Multiple Manipulations ###

The next 2 touch gestures we are going to look at is performing a Scale and Rotation gesture using 2 or more touch inputs. Both of these gestures can be programmed with following the same format as the basic single touch translation above but with a little extra code.

#### Scale ####
{% highlight C# linenos %}
protected override void OnManipulationDelta(ManipulationDeltaEventArgs e)
{
    Point center = new Point(
        BasicRect.RenderSize.Width / 2.0,
        BasicRect.RenderSize.Height / 2.0);

    scale.CenterX = center.X;
    scale.CenterY = center.Y;

    // DeltaManipulation stores the delta data for us so it allows us to
    // focus on the scale gesture
    scale.ScaleX *= e.DeltaManipulation.Scale.X;
    scale.ScaleY *= e.DeltaManipulation.Scale.Y;
}
{% endhighlight %}

#### Rotation ####
{% highlight C# linenos %}
protected override void OnManipulationDelta(ManipulationDeltaEventArgs e)
{
    Point center = new Point(
        BasicRect.RenderSize.Width / 2.0,
        BasicRect.RenderSize.Height / 2.0);

    rotation.CenterX = center.X;
    rotation.CenterY = center.Y;

    // DeltaManipulation stores our rotation delta from the last touch
    rotation.Angle += e.DeltManipulation.Rotation;
}
{% endhighlight %}

### Everything Together ###
Putting everything together is super simple, we just apply our delta to the transform all in the same method that we override. A full translation, rotation and scaling delta code looks like this:
{% highlight C# linenos %}
protected override void OnManipulationDelta(ManipulationDeltaEventArgs e)
{
    Point center = new Point(
        BasicRect.RenderSize.Width / 2.0,
        BasicRect.RenderSize.Height / 2.0);

    // rotation code
    rotation.CenterX = center.X;
    rotation.CenterY = center.Y;    
    rotation.Angle += e.DeltaManipulation.Rotation;

    // scale code
    scale.CenterX = center.X;
    scale.CenterY = center.Y;
    scale.ScaleX *= e.DelteManipulation.Scale.X;
    scale.ScaleY *= e.DeltaManipulation.Scale.Y;

    // translation code
    translation.X += e.DeltaManipulation.Translation.X;
    translation.Y += e.DeltaManipulation.Translation.Y;
}
{% endhighlight %}

It is important to note that with each delta we are taking the current value and applying the new value on top of that. In the case of translation we do `+=`. We do this because the delta is a difference from the last touch input not the absolute position.

## Further Reading ##
At this point you should have a working `UserControl` that responds to basic multi-touch inputs. We have covered the basic of touch gestures and how you can take advantage of the multi touch events via the Manipulation Events in WPF when using .NET 4.0 or greater. While this is a simple tutorial to get you started the Manipulation Events are poweful tools to do a lot of customization. Look for my follow up article that dives into deeper usage of the Manipulation Events and how to apply it to your MVVM Framework. 

All the code from this tutorial and Part 2 is available on [GitHub](https://github.com/ahoefling/wpf.manipulation.demo). Go and download the code or fork it to play around with Manipulations [https://github.com/ahoefling/wpf.manipulation.demo](https://github.com/ahoefling/wpf.manipulation.demo)
