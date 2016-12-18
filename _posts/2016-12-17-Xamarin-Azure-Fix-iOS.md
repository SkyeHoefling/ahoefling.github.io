---
layout: post
title: Fix Azure Mobile App Services Template for Xamarin iOS
modified: 2016-12-17 23:30:00
categories: [Xamarin, Azure]
tags: [Mobile, Xamarin.Forms, iOS, Azure App Services, Xamarin, Azure, C#]
share: true
comments: true
---
Azure Mobile App Services sets you up with a nice template builds all of your boiler plate code. When I went through the setup on the Azure Portal and downloaded the template locally I ran into all sorts of issues with iOS not working in my Xamarin.Forms project. Everything appears to work without issue on Android but I couldn't get the `MobileServiceClient` to load, the app would just crash. After lots of digging and playing with the tools provided I figured out what was wrong. It appears the template was missing some platform specific code for iOS.

Make sure you properly initialize it in the iOS specific project in the AppDelegate.cs. Add the folowing snippet prior to launching the app.

New Code:
{% highlight c# linenos %}
CurrentPlatform.Init();
{% endhighlight %}

Example `AppDelegate.cs`
{% highlight c# linenos %}
[Register("AppDelegate")]
public partial class AppDelegate : global::Xamarin.Forms.Platform.iOS.FormsApplicationDelegate
{
    public override bool FinishedLaunching(UIApplication app, NSDictionary options)
    {
        global::Xamarin.Forms.Forms.Init();
        // new code to properly initialize iOS for Azure
	CurrentPlatform.Init();
        LoadApplication(new App());

        return base.FinishedLaunching(app, options);
    }
}
{% endhighlight %}

Your Xamarin app should now be working with your Azure App Services.
