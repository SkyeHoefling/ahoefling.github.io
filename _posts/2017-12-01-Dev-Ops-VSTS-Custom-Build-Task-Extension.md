---
layout: post
title: Creating a Custom Build Task in VSTS (Visual Studio Team Services)
modified: 2017-12-01 08:00:00
categories: DevOps
tags: [VSTS, Extension, Powershell, Build]
share: true
comments: false
---
Automating builds with VSTS saves the teams countless hours of debugging when someone gives the code the "works on my machine" seal of approval. The tests may not run, there will be stale code between developers and it just causes problems. 


![works on my machine seal]({{ site.url }}/assets/posts/2017-12/works-on-my-machine-seal.png)

The VSTS ecosystem provides a wide set of tools that you should be using. If you aren't head over to [Visual Studio Team Services](https://www.visualstudio.com/vso/) and take a look. 

It is easy to add a new CI build that runs unit tests, VSTS has several templates for most of the builds you want to run that include all the build tasks. Sometimes you have requirments for your build that just don't fit perfectly into the pre-defined VSTS Build Tasks. The best place to start is the VSTS Marketplace where you will find different extensions and build tasks that you can use. You may get lucky and find just what you need but you may need to make something custom and proprietary or something that you will be sharing with the community.

## Overview ##
After going through this article you should be able to do the following:

* Create your very own VSTS Build Task
* Publish your new VSTS Build Task to the marketplace and share it with VSTS instances
  * Anyone can publish to the marketplace but to have public Tasks you need to be a verified VSTS Publisher
* Understanding of the VSTS Extension Ecosystem