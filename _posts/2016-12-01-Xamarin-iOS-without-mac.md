---
layout: post
title:  "How To Develop iOS Without a Mac"
date:   2016-12-01 08:35:00
categories: Xamarin
tags: [iOS, Xamarin.Forms, Xamarin, Apple Developer, bitrise.io, C#, .NET]
share: true
comments: true
---
You have decided to take your first stab at iOS development and you do not have a physical Mac machine running OS X but you have a device running iOS. You can still develop for iOS while you wait for that mac mini to come in the mail. After going through this process and developing in it I don't recommend anyone to develop in this unless it is temporary while you wait for you mac to arrive. 

## Apple Developer Account ##

[Developer Apple](https://developer.apple.com/)

The first thing you will need to do is create your Apple Developer Account, there is no way around this unfortunately. Once you have created the account apple will have to confirm it which can take a few minutes to a few hours. Follow the how-to guides below in the order of their sections to build your necessary profiles and certificates needed to build and deploy.

### Device Registration ###
Before we can do anything we need to register our physical iOS device

1. Navigate to [https://developer.apple.com/account/ios/certificate/](https://developer.apple.com/account/ios/certificate/)
2. Navigate to Devices -> All
3. Click the '+' icon to start the add device wizard
4. UDID is your device's unique identifier, this is easily obtainable through itunes or one of the many free iOS apps out there. If you still can't find your UDID skip all of the items in the Apple Developer Accounts and come back after you complete everything else. In the bitrise.io guide you can easily access your UDID after your first build.
5. Click continue and you have registered your iOS device

### Identifier Registration ###
Before we can create any profiles or certificates we need to create an App ID. 

1. Navigate to [https://developer.apple.com/account/ios/certificate/](https://developer.apple.com/account/ios/certificate/)
2. Navigate to Identifies -> App IDs
3. Click the '+' icon to start the Registering an AppID Wizard


### Certificate ###
Follow the steps below to create your Certificate

1. Navigate to [https://developer.apple.com/account/ios/certificate/](https://developer.apple.com/account/ios/certificate/)
2. Navigate to Certificate -> All
3. Click the '+' icon to start the Create Certificate Wizard
4. Select iOS Development and click continue
5. The next page will tell you how to create a CSR via OS X, we will create this manually. Click continue
6. Open up your favorite terminal application or utility that can run the command `openssl`. 
7. Enter the following commands 
  - This will create the `certSigningRequest` which you will upload to Applie
   {% gist ahoefling/67ea691d210b5a1b1f1333ad26e78e15 %}
8. Back in your browser where it is asking for your `.CSR`/`.certSigningRequest`, upload the new file we just generated and click continue.
9. You have now successfully generated your certificate, download this certificate because you will need it later.

### Provisioning Profile ###
Follow the steps below to create your Provisioning Profile

#### pre-reqs ####
* Device Registration
* Identifier Registration
* Certificate

#### How To Guide ####
1. Navigate to [https://developer.apple.com/account/ios/certificate/](https://developer.apple.com/account/ios/certificate/)
2. Navigate to Provisioning Profiles -> All
3. Click the '+' icon to start the Create Provisioning Profile Wizard
4. Select iOS Development and click continue
5. Select the App ID we created earlier and click continue
6. Select the certificate we created earlier and click continue
7. Select the device we registered and click continue
8. Enter the Provisioning Profile Name you would like to use and click continue
9. You have successfully created a Provisioning Profile, download the profile because you will need it later

### .p12 Strong Signed Certificate ###
To properly deploy your app onto your device you will need a strong signed certificate with an extension of `.p12`

1. Open up a terminal or your utility that has access to `openssl`
2. Navigate to the directory where you saved your `ios_development.cer`
3. Enter the following command
  - This will create the strong signed certificate `.p12`
  - This will prompt you for a password, make sure you remember it because we will need it later
  {% gist ahoefling/3228be1e1f42dbb86b2a196d4309c9be %}
4. You now have successfully created your `.p12` file, save this for later

## Xamarin Project ##
Configure your Xamarin project to work with Android and iOS. It is recommended to use Xamarin.Forms for this so you can test/develop in the android simulator then verify those changes on your iOS device. Otherwise your development workflow will be very slow for each change you would have to try something, commit, build and deploy. (You do have that mac mini in the mail so it shouldn't be too bad, but we used Xamarin.Forms).

For the purposes of the project setup we are going to assume you have Xamarin installed

1. Open Visual Studio (2015 or Xamarin Supported IDE)
2. Click New Project
3. Select Templates -> Visual C# -> Cross-Platform -> Xamarin.Forms (Android/iOS)
  - You can build the application with other UWP/Win8.1/WP8.1 but you need a solution file with just Android/iOS
4. Import your solution into the source control engine that is supported by [BITRISE.io](https://www.bitrise.io/)
  - We used Visual Studio Team Services - git

Feel free to test your Xamarin.Forms solution with the Android Emulator, that goes outside the scope of this post.

## BITRISE.io ##
[BITRISE.io](https://www.bitrise.io/) is a free build agent if you are a team of 1-2 with a focus on mobile app builds and deployments. Since we have our code in source control we are ready to attempt a build and load download our app.

### Register your iOS Device ###
1. Navigate to [BITRISE.io](https://www.bitrise.io/) and create a new account
2. Navigate to your account settings [https://www.bitrise.io/me/profile](https://www.bitrise.io/me/profile)
3. In the left bar menu navigate to 'Test Devices'
4. Register your device via 1 of the 2 options below
  - Navigate here from your iOS device in the safari browser, once here there will be a button to add this device
  - If on your desktop you can register your device manually by clicking the 'Register Manually' button. Have your iOS device's UDID handy because you will need to eneter it.

### Create App ###
1. Navigate to the dashboard [https://www.bitrise.io/dashboard](https://www.bitrise.io/dashboard)
2. Click the '+ ADD' button to create a new app
3. Follow the steps to connect to your code repository

### Add Provisioning Profile and Certificate ###
If you were unable to create a provisioning profile or certificate because you don't know your UDID jump ahead to the build and deploy and come back when you have successfully created the profile and certificate

1. Naviagte to your new apps dashboard and select 'Workflow'
2. In the Workflow Editor select 'Code signing & Files'
3. Under 'Provisioning Profile' select 'Add Provisioning Profile'
  - Add the Provisioning Profile we created earlier `.mobileprovision`
4. Under Code Signing Identity select 'Add Certificate Private Key'
  - Add our `.p12` file we generated earlier, it will ask you for the password you entered earlier

### Build and Deploy ###
We have completed all or almost all of the steps and it is time to build the app

1. Navigate to your apps dashboard
2. Click the 'Build' button and queue a new build.
3. Once the build completes successfully it will send you an email with deployments for the Droid and iOS app.
4. Inside the build completed email there is a button labeled 'install' make sure you open this link from your iOS device and in safari.
5. Once at the deployment link's location you will click the install button and it will attempt to install the app
  - If you were unable to create the provision profile and certificate you will see your UDID here. Copy your UDID and complete the Profile and Certification creation process
6. You can now play with your iOS app that you created without having a mac, way to go!

## References ##
This information was compiled through a night filled with searching and below you will find useful links that I used to put together this guide.

* [http://apple.stackexchange.com/questions/108909/create-key-certificate-signing-request-p12-file-and-provisioning-file-on-ubunt](http://apple.stackexchange.com/questions/108909/create-key-certificate-signing-request-p12-file-and-provisioning-file-on-ubunt)
* [http://stackoverflow.com/questions/9648478/apple-certificate-signing-request](http://stackoverflow.com/questions/9648478/apple-certificate-signing-request)
* [https://patrickshuff.com/generatingsigning-ios-development-keys-on-linux-with-phonegap-build.html](https://patrickshuff.com/generatingsigning-ios-development-keys-on-linux-with-phonegap-build.html)
