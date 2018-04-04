---
layout: post
title:  Push Notifications with AppCenter
modified: 2018-03-23 17:00:00
categories: Xamarin
tags: [Xamarin, Xamarin.Forms, Push, Notifications, AppCenter]
share: true
comments: true
---
Recently I was working on a Xamarin.Forms app that required push notifications for both iOS and Android. I started down the Azure Notification Hub path and was recommended by my friend [James Montemagno](https://twitter.com/JamesMontemagno) that [AppCenter](https://appcenter.ms) supports push notifications. James quickly produced an amazing [blog post](https://montemagno.com/push-notification-options-for-xamarin/) that documents push notification and your options in Xamarin.

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">I need an Easy Button, I spent 5-10 minutes and I was able to get push notifications working with <a href="https://twitter.com/VSAppCenter?ref_src=twsrc%5Etfw">@VSAppCenter</a> and I am now a believer in the technology. I can also easily configure some pretty custom scenario where I can send a push notification to 1 device</p>&mdash; Andrew Hoefling (@andrew_hoefling) <a href="https://twitter.com/andrew_hoefling/status/975208857722114048?ref_src=twsrc%5Etfw">March 18, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## The Push Notification Problem ##
We are building a mobile app and we want to send push notifications, but each platform we are building uses a different push notificaiton server

| Platform| Service                         |
|---------|---------------------------------|
| iOS     | Apple Push Notification Service |
| Android | Firebase Cloud Messaging        |
| Windows | Windows Notification Service    |

Suppose we had a simple service we can reference in our project that handles sending notifications out to our devices instead of having to write custom platform specific code potentially 3 times over. 

## Enter App Center ##
Microsoft's App Center now provides support for handling Push Notifications for your mobile app. They even build out some awesome libraries for you to use in your Xamarin.Forms application. Now I can handle 90% of the work I want to for Push Notifications in my shared code.

### Configure Device Services ###
Before you go any further you will need to configure your platform specific notification services. Head over to the [docs](https://docs.microsoft.com/en-us/appcenter/sdk/push/xamarin-forms) and get your platform specific services configured and then come back

## Configuring App Center ##
Just like most App Center integrations they have provided awesome detailed steps to configuring everything right in the App Center portal

### iOS Specific ###

![App Center Push SDK iOS]({{ site.url }}/assets/2018-03/AppCenter-Push-Add-SDK-iOS.png)

Configuring iOS is really as simple as copying over the details from the APN certificate that you have created. They even provide documentation on this screen on how to do that and approprate links.

![App Center Push APN Config]{{ site.url }}/assets/2018-03/AppCenter-Push-APN.png)

That's it! Yes you read that correctly there is nothing else we need to do to configure iOS

### Android Specific ###
Now that iOS is taken care of you will want to configure your Android Push notification which is in a separate App Center App. Head over there and open up the Push Notification Wizard.

The SDK screen is almost identical between the 2 platforms but there is well documented differences, just be sure to read everything on the SDK configuration screen.

![App Center Push SDK Android]({{ site.url }}/assets/2018-03/AppCenter-Push-Add-SDK-Android.png)

Log into Firebase and get the following properties

* Sender ID
* Server Key

![App Center Push SDK Android Firebase]({{ site.url }}/assets/2018-03/AppCenter-Push-Firebase.png)

![App Center Push SDK Android Server Keyy]({{ site.url }}/assets/2018-03/AppCenter-Push-Android-Server-Key.png)

## Shared Code Xamarin.Forms ##
Yes, we are done configuring App Center! We are ready to start receiving push notifications.

Add the following code before you invoke `AppCenter.Start()`

{% highlight c# linenos %}
if (!AppCenter.Configured)
{
    Push.PushNotificationReceived += (sender, e) =>
    {
        // handle notification code
    }
}
{% endhighlight %}

## Test Push Notification ##
Now everything is setup, let's head back to the App Center Push Notification Portal and fire a test off.

**When testing push notifications it is best to do this on a physical device, you may run into issues on the emulators**

1. Find the blue button that says "Send notification"
2. Fill out the notification Wizard
3. Select your devices
4. Send Notification

You should see the notification alert pop up. Upon clicking the notification if the app is not running it will fire off the event we mentioned aboved. There are some edge cases here depending on which platform you are on. For example I found that the app won't display the alert if your iOS app is running. 

Some things you can do with the notification
* Display local notification
* Run some code
* Update data on the screen
* Whatever you want

## Device Specific Notification ##
This is all well and good but you may not want to just send notifications from the App Center Portal. Automating this is super easy because our friends over at the App Center Team have provided a very powerful API we can use.

Head over to the [AppCenter Swagger Api](https://openapi.appcenter.ms/) and look for the "push" section

![App Center Push API]({{ site.url }}/assets/2018-03/AppCenter-Push-API-Swagger.png)

If you want to send a notification you will want to use 

* `/v0.1/apps/{owner_name}/{app_name}/push/notifications`










