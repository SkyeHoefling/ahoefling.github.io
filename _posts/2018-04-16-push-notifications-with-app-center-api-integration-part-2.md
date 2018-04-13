---
layout: post
title:  Push Notifications with App Center - API Integration
modified: 2018-04-16 08:00:00
categories: Xamarin
tags: [Xamarin, Xamarin.Forms, Push, Notifications, AppCenter, SDK, API]
share: true
comments: true
---
AppCenter Push Notifications is an exciting new technology for handling Push Notifications in any mobile app. I use AppCenter quite a bit with my Xamarin projects so using AppCenter was a natural choice.

This is <strong>Part 2</strong> in a 3 part blog series discussing AppCenter Push Notifications

| Title                                              | Description                                                           |
|----------------------------------------------------|-----------------------------------------------------------------------|
| [Push Notifications with App Center](https://www.andrewhoefling.com/xamarin/2018/03/25/push-notifications-with-app-center.html)                 | The Basics and Configuration                                          |
| Sending Push Notifications with the App Center API | How to trigger a Push Notification from code using the App Center API |
| Local Notifications with App Center                | How to trigger local notifications from a push notification           | 

<br/>

## Enter App Center SDK ##
Sending push notifications to all users or specific Audeniences in the AppCenter portal is a really nice utility for a global or Audenience specific groups. This is not useful when users may be communicating with each other in the app. Consider the classical example of building an email app, when you receive an email you would like to be notified. In this example as part of our email processing logic we can send a command to AppCenter to fire notifications to all of our registered devices. This can be done at the client level or server level depending on your business rules.

![App Center Push API]({{ site.url }}/assets/posts/2018-03/AppCenter-Push-API-Swagger.png)

* [AppCenter Swagger Api](https://openapi.appcenter.ms/) - look for the "push" section

### Swagger Tests ###
Before you start writing any code it is a good idea to run some tests with the SDK and Swagger API provided by the App Center Team. Before we can execute any of the requests we need to set up our authorization access, which can be used in your app or backend system.

1. Open your browser and navigate to [AppCenter.ms](https://appcenter.ms) and log in
2. Once you login you will see your name and avatar at the bottom left of the screen
3. Select Account Settings -> API Tokens

![App Center API Tokens]({{ site.url }}/assets/posts/2018-04/AppCenter-API-Tokens.png)

Select the New API token" button at the top corner of your screen and fill out the form to create your new API Token

* Description
* Full Access (I use this setting when using the SDK API)

![App Center New API Token]({{ site.url }}/assets/posts/2018-04/AppCenter-New-Token-Form.png)

After you create your new token you have <strong>ONE</strong> chance to copy it and write it down. If you forget the token or lose it you will have to generate a new one and update everything that may use that token.

Now that we have the token head over to the [AppCenter Swagger API](https://openapi.appcenter.ms/). We need to authorize our test session

1. Click the big Authorize button
2. Fill out the form with your API Token

![App Center API Swagger Authorize]({{ site.url }}/assets/posts/2018-04/AppCenter-API-Swagger-Authorize.png)

The Swagger API has a lot of different APIs, I find it easiest to hit `ctrl-f` (find) and search for the Push header, see screenshot above. Let's create a new push notification!

1. Select POST - /push/notifications
2. WAIT
3. This page has a lot of APIs so there is sometimes quite a delay when using the API, navigating or typing into fields
4. The test harness should open up, and select the button "Try it out"
5. Now we get to update our JSON, owner_name and app_name

Start by navigating to your app in AppCenter, we can pull all the information out of the address bar:

Here is a list of sample addresses for an app that targets the 3 main platforms

* `https://appcenter.ms/orgs/Hoefling-Software/apps/Sample-App-Android`
* `https://appcenter.ms/orgs/Hoefling-Software/apps/Sample-App-iOS`
* `https://appcenter.ms/orgs/Hoefling-Software/apps/Sample-App-UWP`

| Property           | Description                                                                          |
|--------------------|--------------------------------------------------------------------------------------|
| owner_name         | The name of your organization, in my sample address above we use `Hoefling-Software` |
| app_name (android) | The name of your android app, in my sample address above we use `Sample-App-Android` |
| app_name (iOS)     | The name of your iOS app, in my sample address above we use `Sample-App-iOS`         |
| app_name (UWP)     | The name of your UWP app, in my sample address above we use `Sample-App-UWP`         |

<br />
<strong>WAIT</strong> one second! Yes you are reading that correct, we need to specify a different `app_name` parameter if we want to send requests to an iOS, Android or UWP device. We will go over how we handle this later in code.

This json is built from using the [AppCenter SDK Docs](https://docs.microsoft.com/en-us/appcenter/push/pushapi)

POST Body
{% highlight json linenos %}
{
    "notification_target": {
        "type": "devices_target",
        "devices": [
            "00000000-0000-0000-0000-000000000001",
            "00000000-0000-0000-0000-000000000002", 
            "00000000-0000-0000-0000-000000000003"
        ]
    },
    "notification_content": {
        "name": "Sample Push",
        "title": "Push Title",
        "body": "This is a sample push notification",
        "custom_data": {}
    }
}
{% endhighlight %}

A brief description of each property

| Property                           | Description |
|------------------------------------|-------------|
| `notification_target.type`         | Specifies the type of notificaiton, using the `devices_target` allows us to specify each device we want to push to ||
| `notification_target.devices`      | Specifies an array of physical devices to push to ||
| `notification_content.name`        | The name of the notification, is useful for notification lookup later |
| `notification_content.title`       | The title of the notification that the user will see |
| `notification_content.body`        | The message or body of the notification that the user will see |
| `notification_content.custom_data` | Any custom data you want to send along with the notification that may be processed by the device |

<br />

Execute your request and you should receive it over on your device.

### It Doesn't Work! ###
You followed the steps above and you aren't receiving the push notification on the device even though the API is working.

* <strong>Push notification only work on physical devices and you will run into all sorts of issues with the emulators</strong>
* <strong>Push notifications do not work well when the debugger is attached or was previously attached. If you deployed your app via debugger make sure you kill the app, launch it again and kill it. Then you should be able to receive your notifications</strong>

### Where do I find the Device ID ###
To quote the docs

{% highlight xml linenos %}
The App Center SDK creates a UUID for each device once the app 
is installed. This identifier remains the same for a device when 
the app is updated and a new one is generated only when the app 
is re-installed. The following API is useful for debugging 
purposes.
{% endhighlight %}

Using the api we can get our installId

{% highlight c# linenos %}
System.Guid? installId = await AppCenter.GetInstallIdAsync();
{% endhighlight %}

* [Device Id Docs](https://docs.microsoft.com/en-us/appcenter/sdk/other-apis/xamarin)

## Data To Track ##
Before we can begin building out a system in code that sends push notifications we need to be aware of data we have to track:

* User Information such as a UserID
* DeviceID - a user may have 6 devices or you may want to constrain 1 device per user
* Device Platform - A user may have a device for iOS, Android, etc. You will need to track that data

For the purposes of this article we are going to assume you are properly storing that information and we are able to retrieve all the devices and platforms needed at runtime from our database.

## My Xamarin.Forms Solution ##
This is a breakdown of my Xamarin.Forms solution which boils down to C# code that can be used in Xamarin.Forms or maybe a backend service.

### Models ###
Utilizing [Json.Net](https://www.newtonsoft.com/json) we can define what our C# classes will map to when we create our JSON request.

<strong>Push</strong>
{% highlight c# linenos %}
[JsonObject]
public class Push
{
    [JsonProperty("notification_target")]
    public Target Target { get; set; }

    [JsonProperty("notification_content")]
    public Content Content { get; set; }
}
{% endhighlight %}

<strong>Target</strong>
{% highlight c# linenos %}
[JsonObject]
public class Content
{
    [JsonProperty("name")]
    public string Name { get; set; }

    [JsonProperty("title")]
    public string Title { get; set; }
    
    [JsonProperty("body")]
    public string Body { get; set; }
    
    [JsonProperty("custom_data")]
    public IDictionary<string, string> Payload { get; set;}
}
{% endhighlight %}

<strong>Content</strong>
{% highlight c# linenos %}
[JsonObject]
public class Target
{
    [JsonProperty("type")]
    public string Type { get; set; }

    [JsonProperty("devices")]
    public IEnumerable<string> Devices { get; set; }
}
{% endhighlight %}

Note that we matched up all the JSON names with our same request body from before using the `[JsonProperty]` attribute.

### Notify ###
Here is a modified code sample that I use in production today:

Let's first create some constants:

{% highlight c# linenos %}
public class Constants
{
    public const string Url = "https://api.appcenter.ms/v0.1/apps/";
    public const string ApiKeyName = "X-API-Token";
    public const string ApiKey = "{Your App Center API Token}";
    public const string Organization = "{Your organization name}";
    public const string Android = "{Your Android App Name}";
    public const string IOS = "{Your iOS App Name}";
    public const string DeviceTarget = "devices_target";

    public class Apis
    {
        public const string Notification = "push/notifications";
    }
}
{% endhighlight %}

Don't forget your API Header! I am not documenting making the actual RESTful call, but don't forget to plug in:

* Your API Token in the header
* Your content type
* Your method type if applicable

{% highlight c# linenos %}
public async Task Notify(
    User user, 
    string name, 
    string title, 
    string body, 
    IDictionary<string,string> payload)
{
    // let's assume you have a User object that contains
    // * iOS Devices
    // * Android Devices

    var push = new Push
    {
        Content = new Content
        {
            Name = name,
            Title = title,
            Body = body,
            Payload = payload
        },
        Target = new Target
        {
            Type = Constants.DeviceTarget
        }
    };

    if (user.IOSDevices.Any())
    {
        push.Target.Devices = user.IOSDevices;
        await PostAsync(
            $"{Constants.Organization}/{Constants.IOS}/{Constants.Apis.Notification}", 
            JsonConvert.SerializeObject(push));
    }

    if (user.AndroidDevices.Any())
    {
        push.Target.Devices = user.AndroidDevices;
        await PostAsync(
            $"{Constants.Organization}/{Constants.Android}/{Constants.Apis.Notification}", 
            JsonConvert.SerializeObject(push));
    }
}
{% endhighlight %}

<strong>NOTE</strong> We only send the iOS devices to the iOS App and the Android devices to the Android app.

The code above will send push notifications to all devices that may be registered for a user. Now it is up to the app on those devices to properly display the notification or run some logic such as 2 way data binding.

## Local Notifications ##
Push Notifications are a powerful way to notify the user that something has happened. If you have been following along up until this point you may notice your push notifications are not working while the app is running. The final article in this blog series will document how to handle local notifications.