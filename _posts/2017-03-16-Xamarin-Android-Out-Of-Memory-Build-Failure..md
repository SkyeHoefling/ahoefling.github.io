---
layout: post
title:  Xamarin Android Out of Memory Build Failure
modified: 2017-03-16 08:00:00
categories: Xamarin
tags: [Xamarin, Xamarin.Forms, Android, Java, OutOfMemoryException, Build, Failure, Memory, Heap, Size, Manifest, largeHeap]
share: true
comments: true
---
In Xamarin.Forms or Xamarin Android your project might get to a point where you run into issues with the java build failing for what appears to be no reason. At first glance the `OutOfMemoryException` may make no sense at all, but toggling some simple settings will get you back up and running.

Truncated Error
{% highlight xml linenos %}
C:\Program Files (x86)\MSBuild\Xamarin\Android\Xamarin.Android.Common.targets(2072,3): 
Error XA5213: java.lang.OutOfMemoryError. 
Consider increasing the value of $(JavaMaximumHeapSize). 
Java ran out of memory while executing 'java.exe...'
{% endhighlight %}

Below you will find different potential fixes

* Increase the Heap Size
* Allow Large Heap
* Build with Java x64 instead of x86

## Increase the Heap Size ##
Increasing the Heap Size should be the simple solution to fix this problem. When you create your Xamarin Android project it leaves the Heap Size blank by default. Updating this value to `1G` may resolve your problem. 

In my experience I have not been able to update the Heap Size to `2G`. Whenever I do this I continue to get build errors, even if it was working with `1G`. You will have to change this for each build configuration that is failing. In our example here we fix it for `Debug|AnyCPU`.

### Visual Studio ###
1. Right click on the Android Project and select `Properties`
2. Navigate to `Android Options` -> `Advanced`
3. Update Heap Size to `1G`

![Java Max Heap Size]({{ site.url }}/assets/posts/2017-03/Java-Max-Heap-Size.png)

### XML .csproj ###
Add the following code to your .csproj file inside your build `<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">`

{% highlight xml linenos %}
<JavaMaximumHeapSize>1G</JavaMaximumHeapSize>
{% endhighlight %}

## Allow Large Heap ##
We can specify in the `AndroidManifest.xml` file to allow a Large Heap. This setting will allow the compiler to work when we are still getting the `OutOfMemoryException` even after updating the Heap Size to `1G`.

Navigate to your Android Manifest file located inside your android project -> properties -> `AndroidManifest.xml`.

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.mykpaonline.mykpaonsite" android:installLocation="auto" android:versionCode="1" android:versionName="1.0">
	<uses-sdk android:minSdkVersion="22" />
  <application android:largeHeap="true" />
</manifest>
{% endhighlight %}

## Build With Java x64 instead of x86 ##
If you are still running into issues with with the build failing with an `OutOfMemoryException` you can try upgrading your JDK to use x64 instead of x86. The x64 JDK handles the larger Heap Size better than the x86 JDK.