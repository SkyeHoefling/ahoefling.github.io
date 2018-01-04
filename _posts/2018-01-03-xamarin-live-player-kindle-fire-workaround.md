---
layout: post
title:  Xamarin Live Player Kindle Fire Workaround
modified: 2018-01-03 22:30:00
categories: Xamarin
tags: [LivePlayer, Android, Kindle, Device, Workaround]
share: true
comments: true
---
Let's talk about using your Kindle Fire as a development device for Xamarin. It is running Android OS so you should be able to develop and test your apps on it just like any other android device. I was able to get the Xamarin Live Player working on my Kindle Fire with a simple workaround since the regular pairing was not working.

Recently I started configuring an old Kindle Fire to use as a development and testing device for my Xamarin projects. The first thing I tried was downloading the Xamarin Live Player app through HockeyApp to connect to Visual Studio. The app appeared to be working correctly but after scanning the QR Code from Visual Studio I quickly realized this was not going to work.

The Xamarin Live Player may work out of the box on your Kindle Fire and with your configuration, but if you are running into the same problem I ran into keep reading!

## System Specs ##

* Windows 10
* Visual Studio 2017 15.5.0 Preview 4.0
* Xamarin Live Player 1.3.117 (617)

## Download Xamarin Live Player ##
If you do not have the latest build of Xamairn Live Player, go get it. Goto the [Xamarin Live Player docs](https://developer.xamarin.com/guides/cross-platform/live/install/#1._Get_the_App) and select the HockeyApp distrubtion. You will be prompted to log in and then can download the `APK` which you can then install on your device.

## Configure Visual Studio ##
Go ahead and try to connect your Kindle Fire to Visual Studio via the Xamarin Live Player, if you are lucky it will work but if you are like me it will not work.

With help from James Montemagno there is a workaround documented for a similar issue with Android 8.1 in the [Xamarin Forums](https://forums.xamarin.com/discussion/113879/android-8-1-not-showing-in-debug-list-workaround#latest). This workaround is based completely off of this article and with a little luck.

### The Workaround ###

1. Navigate to `%userprofile%\AppData\Roaming`.
2. Open `PlayerDeviceList.xml`
3. Add a new Device Entry for your Kindle Fire
4. Restart Visual Studio
5. Deploy your app via Xamarin Live Player

#### Device Entry ####
Remove the comments below before saving
{% highlight xml linenos %}
<PlayerDevice>
	<!-- Secret code from Xamarin Live Player App. This is the code that you use for manual sync -->
	<SecretCode>XXXXX</SecretCode>

	<!-- a Guid, this can be any unique identifier -->
	<UniqueIdentifier>48e67d1e-9915-4af0-a704-dc0ac6771b41</UniqueIdentifier>

	<!-- The name you want to give your Device to use in Visual Studio -->
	<Name>Kindle Player</Name>

	<!-- Kindle Fire is a fork of Android so we speciy platform as Android -->
	<Platform>Android</Platform>

	<!-- You will need to determine your API version which you can find from the fork OS version. Using this article https://en.wikipedia.org/wiki/Fire_OS you can determine what android OS base you are using then get the API version from that. I am using Fire OS 5.6.0.0 which is built from android 5.1.1 that supports API 20. -->
	<AndroidApiLevel>20</AndroidApiLevel>

	<!-- This is the device IP Address you can get this from the connection test of the Xamarin Live Player -->
	<DebuggerEndPoint>192.168.1.123:37847</DebuggerEndPoint>

	<!-- the rest of the settings were just copied from another device I had already configured -->
	<HostEndPoint />
	<NeedsAppInstall>false</NeedsAppInstall>
	<IsSimulator>false</IsSimulator>
	<SimulatorIdentifier />
	<LastConnectTimeUtc>2017-12-31T20:15:23.4805003Z</LastConnectTimeUtc>
</PlayerDevice>
{% endhighlight %}

#### Complete PlayerDeviceList.xml ####
Here is a complete xml file with just our Fire Device

{% highlight xml linenos %}
<DeviceList xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
	<Devices>
		<PlayerDevice>
			<SecretCode>XXXXX</SecretCode>
			<UniqueIdentifier>48e67d1e-9915-4af0-a704-dc0ac6771b41</UniqueIdentifier>
			<Name>Kindle Player</Name>
			<Platform>Android</Platform>
			<AndroidApiLevel>20</AndroidApiLevel>
			<DebuggerEndPoint>192.168.1.123:37847</DebuggerEndPoint>
			<HostEndPoint />
			<NeedsAppInstall>false</NeedsAppInstall>
			<IsSimulator>false</IsSimulator>
			<SimulatorIdentifier />
			<LastConnectTimeUtc>2017-12-31T20:15:23.4805003Z</LastConnectTimeUtc>
		</PlayerDevice>
	</Devices>
</DeviceList>
{% endhighlight %}

## Workaround Notes ##
This is only a workaround and it may not work on your device/dev environment