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

## Download the CLI ##
To get started you will need to download the CLI using node.js, if you don't have version 4.0 or greater head over to [https://nodejs.org/](https://nodejs.org/) and download the latest version.

Execute the following command from your favorite shell/command prompt
`npm install -g tfx-cli` 

You should now have the CLI successfully installed and you are ready to start developing your new VSTS Build Task.

## Build Scaffolding ##
There is no quick scaffolding command that I found in the Visual Studio docs but there is some in-depth tutorials that you will find useful. (see useful links at the bottom).

In your root directory create the following structure and files

{% highlight text linenos %}
|- README.md
|- vss-extension.json
|- icon.png
|- buildtasks
    |- task.json
    |- powershell.ps1
{% endhighlight %}

### README.md ###
This is really just for you, it is a `README.md` file. You should document what your VSTS Build Task is going to do and it would be cool if you hosted it on GitHub to contribute to the community. Unless you need to keep it in house. Either way document what your task does, it is important.

### vss-extension.json ###
The `vss-extension.json` tell the `tfx` how to build your extension. This file contains all of your metadata and configuration. Some important sections that we have to fill out:

* `id` - This is the camalCaseName of the task
* `version` - The current version of the task
* `name` - The friendly name of the task
* `description` - The description of the task, which will be displayed on the marketplace
* `publisher` - your publisher ID
  * Go to the Visual Studio Market Place publisher to create your publisher profile [https://marketplace.visualstudio.com/manage/publishers](https://marketplace.visualstudio.com/manage/publishers)
* `targets` - What type of extension are you building? In our case it should always be `Microsoft.VisualStudio.Services` for VSTS Build Task
* `icons` - The icon image you want to represent the build task
* `contribution` - contains meta information about the task
* `files` - All the files that compose the build task

{% highlight json linenos %}
{
	"manifestVersion": 1,
	"id": "HelloBuildTask",
	"version": "1.0.0",
	"name": "Hello World",
	"description": "A VSTS Build Task that prints Hello World to the Console",
	"publisher": "hoefling-software",
	"targets": [
		{
			"id": "Microsoft.VisualStudio.Services"
			}
		],
	"icons": {
		"default": "icon.png"
	 },
	"contributions": [
		{
			"id": "HelloBuildTask",
			"type": "ms.vss-distributed-task.task",
			"targets": [
				"ms.vss-distributed-task.tasks"
			],
			"properties": {
				"name": "buildtask"
			}
		}
	],
	"files":[
		{
			"path": "buildtask"			
		}
	]
}
{% endhighlight %}

### icon.png ###
The icon image file that will be used to identify your build task in the marketplace and wherever it is used inside of VSTS

### task.json ###
The Task metadata file which is located in your build task describes the individual task and what it executes. This includes parameters the user needs to enter and which scripts to execute.

Here is a simple `task.json` file that executes a powershell script

{% highlight json linenos %}
{
    "id": "5164728d-cfca-4576-a066-bde89930bf2b",
    "name": "HelloWorld",
    "friendlyName": "Hello World",
    "description": "Prints Hello World to the console",
    "helpMarkDown": "",
    "category": "Build",
    "visibility": [
        "Build"
    ],
    "runsOn": [
        "Agent",
        "DeploymentGroup"
    ],
    "author": "Andrew Hoefling",
    "version": {
        "Major": 0,
        "Minor": 0,
        "Patch": 16
    },
    "instanceNameFormat": "Prints Hello World",
    "groups": [
        {
            "name": "advanced",
            "displayName": "Advanced",
            "isExpanded": false
        }
    ],
    "inputs": [
        {
            "name": "name",
            "type": "string",
            "label": "Name",
            "defaultValue": "",
            "required": true,
            "helpMarkDown": "Enter your name to print to the console"
        }
    ],
    "execution": {
        "PowerShell3": {
            "target": "powershell.ps1",
            "platforms": [
                "windows"
            ],
            "workingDirectory": "$(currentDirectory)"
        }
    }
}
{% endhighlight %}

TODO - we need to add information about each section and a screenshot what this will look like as a build task.

### Buildtasks Powershell Script ###


