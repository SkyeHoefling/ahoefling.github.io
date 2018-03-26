---
layout: post
title:  Push Notifications with App Center
modified: 2018-03-25 23:00:00
categories: Xamarin
tags: [Xamarin, Xamarin.Forms, Push, Notifications, AppCenter]
share: true
comments: true
---
Recently I was working on a Xamarin.Forms app that required push notifications for both iOS and Android. I started implementing Azure Notification Hub and was recommended by my friend [James Montemagno](https://twitter.com/JamesMontemagno) that [App Center](https://appcenter.ms) supports push notifications. James quickly produced an amazing [blog post](https://montemagno.com/push-notification-options-for-xamarin/) that documents push notification and your options in Xamarin.

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en"><p lang="en" dir="ltr">I need an Easy Button, I spent 5-10 minutes and I was able to get push notifications working with <a href="https://twitter.com/VSAppCenter?ref_src=twsrc%5Etfw">@VSAppCenter</a> and I am now a believer in the technology. I can also easily configure some pretty custom scenario where I can send a push notification to 1 device</p>&mdash; Andrew Hoefling (@andrew_hoefling) <a href="https://twitter.com/andrew_hoefling/status/975208857722114048?ref_src=twsrc%5Etfw">March 18, 2018</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<br/>
This will be a 3-Part Blog series on Push Notifications with Visual Studio App Center

| Title                                              | Description                                                           |
|----------------------------------------------------|-----------------------------------------------------------------------|
| Push Notifications with App Center                 | The Basics and Configuration                                          |
| Sending Push Notifications with the App Center API | How to trigger a Push Notification from code using the App Center API |
| Local Notifications with App Center                | How to trigger local notifications from a push notification           | 

<br/>

## The Push Notification Problem ##
We are building a mobile app and we want to send push notifications, but each platform we are building uses a different push notificaiton server

| Platform| Service                         |
|---------|---------------------------------|
| iOS     | Apple Push Notification Service |
| Android | Firebase Cloud Messaging        |
| Windows | Windows Notification Service    |

<br />

Suppose we had a simple service we can reference in our project that handles sending notifications out to our devices instead of having to write custom platform specific code potentially 3 times over. 

## Enter App Center ##
Microsoft's App Center now provides support for handling Push Notifications for your mobile app. They even build out some awesome libraries for you to use in your Xamarin.Forms application. Now we can handle 90% of the work we want for Push Notifications in our shared code.

### Configure Device Services ###
Before you go any further you will need to configure your platform specific notification services. Head over to the [docs](https://docs.microsoft.com/en-us/appcenter/sdk/push/xamarin-forms) and get your platform specific services configured and then come back

## Configuring App Center ##
Just like most App Center integrations they have provided awesome detailed steps to configuring everything right in the App Center portal

### iOS Specific ###

![App Center Push SDK iOS]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-Add-SDK-iOS.png)

Configuring iOS is really as simple as copying over the details from the APN certificate that you have created. They even provide documentation on this screen on how to do thata and approprate links.

![App Center Push APN Config]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-APN.png)

Head over to visual studio and update your `Entitlements.plist`

* Allow Push Notifications

That's it! Yes you read that correctly there is nothing else we need to do to configure iOS

### Android Specific ###
Now that iOS is taken care of you will want to configure your Android Push notification which is typically in a separate App Center App. Head over there and open up the Push Notification Wizard.

The SDK screen is almost identical between the 2 platforms but there is well documented differences, just be sure to read everything on the SDK configuration screen.

![App Center Push SDK Android]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-Add-SDK-Android.png)

Log into Firebase and get the following properties

* Sender ID
* Server Key

![App Center Push SDK Android Firebase]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-Firebase.png)

![App Center Push SDK Android Server Keyy]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-Android-Server-Key.png)

Head over to visual studio and update your `AndroidManifest.xml`

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:versionCode="1" android:versionName="1.0" package="com.hoeflingsoftware.PushSample" android:installLocation="auto">
	<uses-sdk android:minSdkVersion="15" />
	<!-- app center push - START -->
	<permission android:protectionLevel="signature" android:name="${applicationId}.permission.C2D_MESSAGE" />
	<uses-permission android:name="${applicationId}.permission.C2D_MESSAGE" />
	<!-- app center push - END -->
	<application android:label="PushSample" android:icon="@drawable/icon"></application>
</manifest>
{% endhighlight %}

## Shared Code Xamarin.Forms ##
Yes, we are done configuring App Center! We are ready to start receiving push notifications.

Add the following code before you invoke `AppCenter.Start()`

{% highlight c# linenos %}
if (!AppCenter.Configured)
{
    Push.PushNotificationReceived += (sender, e) =>
    {
        // handle notification code
        Debug.WriteLine("Notification Received!!!");
        Debug.WriteLine($"Title: {e.Title}");
        Debug.WriteLine($"Message: {e.Message}");
    }
}
{% endhighlight %}

## Test Push Notification ##
Now everything is setup, let's head back to the App Center Push Notification Portal and fire a test off.

Before you send the notification you may want to head back to visual studio and setup some breakpoints with the app running. 

**When testing push notifications it is best to do this on a physical device, you may run into issues on the emulators**

1. Find the blue button that says "Send notification"
2. Fill out the notification Wizard
3. Select your devices
4. Send Notification

You should receive the notification in your shared code and you can now process it however you want. 

Some things you can do with the notification
* Display local notification
* Run some code
* Update data on the screen
* Whatever you want

Now that we know our app is receiving the push notification we can test it while the app is not running. The real use-case for push notifications. Follow these steps prior to testing the push notification, if you do not follow these steps you will run into false negatives where you won't receive the notification because the debugger is still holding onto the app.

1. Deploy your app via visual studio debugger
2. Stop the debugger
3. Terminate the app
4. Launch the app via device
5. shutdown the app

At this point you are ready to receive some notifications. Go back to App Center and trigger the notifications for both iOS and Android and you should see notifications like these:

![App Center Push - iOS]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-iOS.jpg)
![App Center Push - Android]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-Android.png)


## Device Specific Notification ##
You have a few options from App Center when creating notifications

| Type                   | Description                                                                           |
|------------------------|---------------------------------------------------------------------------------------|
| All Registered Devices | Sends a notification to everyone with your app                                        |
| Custom Device List     | Sends a notification to a specific list you specify                                   | 
| Audience               | Sends a notification to an App Center customized audience, you are allowed 5 for free |

<br />
## Code Sample ##
I have put together a simple Xamarin.Forms app that supports Push Notification. To get this working locally you will need to fork the repository and update the App Center configuration to use your configuration. At that point you should be able to use Push Notifications with App Center.

* [Xamarin Push Sample](https://github.com/ahoefling/XamarinAppCenterPushSample)

## App Center API ##
Next time we will talk about using the App Center API and how to send customized push notifications from code. While you wait for the next article take a look at the api

![App Center Push API]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-API-Swagger.png)

* [AppCenter Swagger Api](https://openapi.appcenter.ms/) - look for the "push" section