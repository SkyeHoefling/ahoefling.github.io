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

| Property    | Description                           |
|-------------|---------------------------------------|
| id          | This is the camalCaseName of the task |
| version     | The current version of the task       |
| name        | The friendly name of the task         |
| description | The description of the task           |
| publisher   | your publisher ID
|             | Go to the Visual Studio Market Place publisher to create your publisher profile [https://marketplace.visualstudio.com/manage/publishers](https://marketplace.visualstudio.com/manage/publishers)|
| targets     | What type of extension are you building? In our case it should always be `Microsoft.VisualStudio.Services` for VSTS Build Task |
| icons       | The icon image you want to represent the build task    |
| contribution| contains meta information about the task               |
| files       | All the files that compose the build task              |


<br />
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

The `task.json` configures what the user is going to see when they select your build task in the UI. Here is an example of what a rendered Hello World build task will look like

![Hello World Build Task UI]({{ site.url /assets/posts/2017-12/Build-Task-UI-Settings.png }})

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

| Property     | Description                                         |
|--------------|-----------------------------------------------------|
| id           | A GUID to uniquely identify the build task          |
| name         | The name of the build task                          |
| friendlyName | The friendly name of the build task                 |
| description  | The description of the build task                   |
| author       | The author of the build task                        |
| version      | The version to display in format (x.x.x)            |
| inputs       | The inputs the user will be able to enter data      |
| execution    | The scripts the build task will execute             |

### Buildtasks Powershell Script ###
If you examine the end of the `task.json` you will notice at the bottom of the file there is an `execution` property. This property defines what scripts will execute.

`task.json` snippet
{% highlight json linenos %}
"execution": {
    "PowerShell3": {
        "target": "powershell.ps1",
        "platforms": [
            "windows"
        ],
        "workingDirectory": "$(currentDirectory)"
    }
}
{% endhighlight %}

This part of the metadata file tells the build task to execute the `powershell.ps1` script in the working directory of build agents current execution.

## Powershell Script and Modules ##
Let's try creating our Hello World script, all we care about right now is printing "Hello World" to the console. See the powershell script below.

{% highlight ps1 linenos %}
[CmdletBinding()]
param()

Write-Host "Hello World"
{% endhighlight %}

If we take everything we have done and deployed it out to our build enviornment we will get "Hello World" printing to the console. If you would like to attemp this skip ahead to the Publishing and Deployment section below, but be sure to come back after you test it.

...

Now that you have tried a publish and deployment you can't really do anything with this unless you can get the parameters from the metadata file `task.json`. That is where the `VstsTaskSdk` Powershell Module comes in. It is a useful module that allows us easy access to the task inputs. When we load the module in you may run into a few issues that we are going to walk through.

Install VstsTaskSdk Powershell Module
1. open up powershell
2. navigate to the `root/buildtask` of directory of your extension
3. execute `mkdir ps_modules` and then navigate into the new directory
4. your `pwd` should read `root/buildtask/ps_modules`
5. execute `Save-Module -Name VstsTaskSdk -Path .` which will save the module to disk.
6. Flatten the directory structure by removing the version number. For example you will have a path of `root/buildtask/ps_modules/VstsTaskSdk/0.10.0/*` which should now read `root/buildtask/ps_modules/VstsTaskSdk/*`

When you are installing Powershell Modules if you don't remove the version folder and flatten it you will run into issues on the build machine when it tries to find the reference. 

Now we have access to the APIs to retrieve information from the `task.json` and we can make a more powerful powershell script.

{% highlight ps1 linenos %}
[CmdletBinding()]
param()

Trace-VstsEnteringInvocation $MyInvocation
try {
    # Get inputs.
    $inputName = Get-VstsInput -Name 'name' -Require

    Write-Host "Hello $inputName"
} finally {
    Trace-VstsLeavingInvocation $MyInvocation
}
{% endhighlight %}

This script retrieves the `name` property we defined as an input and it will print out "Hello Andrew" to the console if you enter the name "Andrew" for the name.

## Publishing and Deployment ##
The code is completed and it is now time to publish our extension to the marketplace and deploy it into our next VSTS Build. This is where everything starts coming together and is how you can test your task.

The first thing you have to do is create your publishing profile so you can upload your Visual Studio Extension `.vsix` into the marketplace and add it to your VSTS instance. Open a browser and navigate to the marketplace to create your publishing profile

[https://marketplace.visualstudio.com/manage/publishers](https://marketplace.visualstudio.com/manage/publishers)

Once you have created a publishing profile you will need to edit the `vss-extension.json` file to use the publisher ID you just created. My publisher id is `hoefling-software` so we are going to use that.

Before generating the extenstion file I make sure my version numbers are in sync with my `task.json` file and `vss-extension.json`. Once you confirm this you can proceed with generating your extension file.

Navigate to the root directory and execute the following command with `tfx-cli`
{% highlight text linenos %}
tfx extension create --manifest-globs vss-extension.json
{% endhighlight %}

This will create a `.vsix` file which is a Visual Studio Extension file. The name will be something like `hoefling-software.HelloBuildTask-1.0.0.vsix` where the name conforms to the standard defined below

| Name              | Property  |
|-------------------|-----------|
| hoefling-software | Publisher |
| HelloBuildTask    | id        |
| 1.0.0             | version   |

<br />
Now that we done all the hard work we can upload our extension to the marketplace. Navigate to the [marketplace](https://marketplace.visualstudio.com/manage/publishers)

1. Click the `Upload new extension` button in green at the top right of the screen
2. You will see a "upload new item" modal appear where you can drag and drop or browse to your `.vsix` file. Upload the file that we just generated from the `tfx-cli`
3. Click `Upload`

You will now see your new extension appearing in the marketplace for your publisher. Until you become a verified publisher you will not be able to see your extension on the public marketplace. We talk about becoming a verified publisher below.

At this point we can share our extension with anyone that wants it. When you hover over your extension you will see the `...` option appear. If you click that and select the share option you can specify the VSTS instance you would like to share your extension with.

Once your extension is shared you can log into that VSTS instance and add the extension via extension page. You will need to be at the root level of VSTS to access extensions for the account. Once you get to the extensions screen you will see a screen like this:

![works on my machine seal]({{ site.url }}/assets/posts/2017-12/extension-page.png)

Select the shared extension and install it. Your extension is successfully installed

## Testing the Build Task ##
If you have gotten this far you have now created, published and deployed your custom Build Task and you want to see it in action. 

1. Open up VSTS and head over to any project and navigate to the Build and Release section
2. Create new build, select empty process
3. Click add task and search for hello world
4. You will see the custom build task, now add it to the build
5. Modify the parameter by plugging in your name
6. save and queue the new build

When you complete the task your output should contain your build task:
![console log]({{ site.url }}/assets/posts/2017-12/build-task-console.png)

You have now successfully created your first VSTS Build Task and got it working in your VSTS instance.

## Verified Publishers and the Public Marketplace ##
If you want to add your new Build Task to the public marketplace you will need to send an email to microsoft and request permission. They have their own set of rules of who can and can not become a verified publisher.

More information on this topic can be found at the marketplace publisher portal.

## Useful Links ##

* [Microsoft Documentation](https://www.visualstudio.com/en-us/docs/integrate/extensions/develop/add-build-task)
* [GitHub Repo of Microsoft Tasks](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks)
* [Code Sample](https://github.com/ahoefling/build-task-demo)