---
layout: post
title:  Xamarin Android Virtualized Development Environment Debug Over Open VPN
modified: 2017-03-27 08:00:00
categories: Xamarin
tags: [Xamarin, Android, VPN, Debug, VM, OpenVPN]
share: true
comments: true
---
As a consultant I work on several different projects throughout a calendar year and I may need to circle back to old projects. It is very useful for me to have a virtual machine I can just boot up to pick up exactly where I left off. This also makes creating a new development enviornment super easy for me, I just spin up a new virtual machine and I am ready to go.

I have ran into several headaches while having a virtualized development enviornment in the mobile world. There are all sorts of DevOps problems you run into from not being able to nest virtualized machines to getting all the networking to play nicely together. 

I do a lot of travelling and I typically bring my work with me which then requires me to have all my networking setup correctly. 

## The Problem ##
Currently I work on a Xamarin.Forms project where I use an Azure Virtual Machine as my development machine and I use my laptop as my Android Emulator.

### Normal Network Diagram ###
Typically when I am working at home I connect my Azure Virtual Machine to my office network via OpenVPN and everything is perfect. I am then able to use the Android Device Bridge to easily connect the machines together for active debugging. 

![Office Network]({{ site.url }}/assets/posts/2017-03/Normal-Network.png)

### Travelling Network Diagram ###
While I am on the road travelling things get a little bit more complicated. I now need to make sure I am able to connect to my home network via Open VPN through my android device.

![Travelling Network]({{ site.url }}/assets/posts/2017-03/Travelling-Network.png)

## The Solution ##
The simple solution is install the VPN client from your android device or emulator and connect back to your VPN. Once this is completed you can easily connect the Android Device to your Visual Studio environment via Android Device Bridge (ADB)

### Pre-Reqs ###
You will need the following pre-reqs prior to starting this how-to guide.

* OpenVPN server
* OpenVPN Client Configuration file `ovpn`

### Guide ###

#### Android Device VPN ####
1. If you are running on a physical Android Device you most likely have Google Apps installed, verify that you do and jump ahead to step X
 * Navigate to [http://opengapps.org/](http://opengapps.org/) select your proper architecture, version and package you would like. I typically use nano variant which contains everything I need.<br />
  ![Open Gapps Select]({{ site.url }}/assets/posts/2017-03/gapps.png)
 * Download the gapps zip package
 * Install the package to your emulator. We use Genymotion and you can just drag and drop the gapps package into the emulator and it will install.
2. Open up the Google Play store, search and install `OpenVPN for Android`.
 ![OpenVPN for Android]({{ site.url }}/assets/posts/2017-03/google-play-openvpn-for-android.png)
3. Launch `OpenVPN for Android` you will have a navigation bar that looks like this:
 ![OpenVPN for Android Navigation]({{ site.url }}/assets/posts/2017-03/vpn-app-landing-page.png)
4. Import your `.ovpn` configuration file to the application
 * Select the `+` icon to start importing a new VPN Configuration
 * On the Import Modal click on the `Import` button at the bottom left
  ![OpenVPN for Android Import Modal]({{ site.url }}/assets/posts/2017-03/vpn-import.png)
 * Navigate to your locally stored configuration file `.ovpn` and select it to import. The application will read the file and properly configure the VPN
5. Set your VPN profile name <br />
  ![OpenVPN for Android Set Profile Name]({{ site.url}}/assets/posts/2017-03/vpn-profile-name.png)
6. You are all set to connect your android device to your OpenVPN, select it from the main page and it will now connect.
7. Open up your android terminal and enter the following commands
 * `su` (you should have your device rooted)
 * `setprop service.adb.tcp.port 5555` (you can specify your own port, but `5555` is the default for adb)
 * `stop adbd`
 * `start adbd`

Now you are all set from the android side of things, follow the Virtual Machine Guide to finish up the process.

#### Virtual Machine VPN ####
Since our Android device is all configured and connected to our VPN it is time to connect the device to our development machine via Android Device Bridge (ADB).

1. Connect your development machine to your VPN Server (we aren't going into detail about how to do this)
2. Open up a command prompt and try to ping your device by ip address
 * If the ping is not successful go back to the android device how-to guide and make sure you did everything correctly
3. Open up the android command prompt which is found at `Tools->Android`
4. A command prompt should appear, enter the following command
 * `adb connect 192.168.1.1:5555` where you enter your android devices IP Address instead of the default IP specified here.

 At this point you should receive a message that you are connected and you should see your device appearing in your debug options. Click debug and start the deploy/debug process.