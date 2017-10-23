---
layout: post
title:  DNN Continuous Integration and Continuous Deployment with VSTS and Azure - Clean Install
modified: 2017-10-23 08:00:00
categories: DevOps
tags: [DNN, VSTS, Build, CI, CD, Azure, Release]
share: true
comments: true
---
[DNN](http://www.dnnsoftware.com/) (DotNetNuke) is a Dot Net based Content Management System (CMS). Building DNN Modules allows you to easily extend the functionality of the DNN Website with complex apps that you can bolt on to any page in your CMS.

After using several DNN Modules I have found that building, releasing and deploying modules can be a very manual process. This is true for modules that are released outside of your organization or modules that are released internally in your website. 

## Installing vs Side-Loading Modules ##
when building a DNN Module when you build the module in release mode you generate a `.zip` installer file. This installer file can be bolted into your website and run necessary install scripts such as SQL scripts. After a project is installed a common approach is side-loading updates, where the administrator just copies over the latest assemblies and code files.

While side-loading your module is an easy way to patch and update your module I find it a more apprioprate solution to use the installer to properly upgrade your module. Using the installer the module will properly update it's version number which will allow the Development and Quality Assurance teams to proplery verify the modules are working as designed.

## Pre-Reqs ##
This tutorial will walk you through installing DNN Community Edition and set up VSTS builds that create versioned installer files and deploys the website to Azure.

| Pre-Req                     | Link                                                                                           |
|-----------------------------|------------------------------------------------------------------------------------------------|
| DNN 9 Platform              | [http://www.dnnsoftware.com/community/download](http://www.dnnsoftware.com/community/download) |
| DNN 9 Templates             | [https://github.com/ChrisHammond/DNNTemplates](https://github.com/ChrisHammond/DNNTemplates)   |
| Visual Studio 2015+         | [https://www.visualstudio.com/](https://www.visualstudio.com/)                                 |
| Visual Studio Team Services | [https://www.visualstudio.com/team-services/](https://www.visualstudio.com/team-services/)     |
| Azure Subscription          | [https://azure.microsoft.com/](https://azure.microsoft.com/)                                   |

## Create Repository for DNN 9 Platform ##
Adding a clean install of the DNN 9 platform to source control is the first step to getting everything working. I recommend creating several branches in your `git` repository which reference the different enviornments. This scaffolding will be useful later on when you want to do production upgrades.

1. Create a new branch in your VSTS instance and call it `website`
2. After creating your repository it is recommended to create a branch for each environment (dev/staging/uat/prod/etc)
3. Clone your repository to your local disk
4. Download the DNN 9 Platform installer and install DNN 9 Platform into your cloned repository
  1. DO NOT run the DNN installer you want the clean website saved in source control
5. Add the entire clean website to git and push your changes up

## Create your DNN Module ##
If you do not have a DNN Module created we need to create it. There are several things we need to modify from the DNN MVC Template to get it to work correctly.

It is important to note that the DNN Module is modified to be built independently from the DNN Website. Your module environment currently may require the DNN Website. To properly follow this guide you will need to de-couple the Website and the Module.

1. Go to your VSTS Instance and create a new Repository within the same project as we did earlier for your DNN Module
2. Clone your repository
3. Create a bin directory at the root and copy over the assemblies from the bin directory of the DNN Website. Add all of these changes to source control
4. Goto [https://github.com/ChrisHammond/DNNTemplates](https://github.com/ChrisHammond/DNNTemplates) and install the latest DNN Visual Studio Template for your version of Visual Studio
5. Open Visual Studio
6. Create New Project
7. Search for the DNN MVC Template and create your new DNN MVC Module
8. Create your new project in your new repository.
9. Unload your `.csproj` if necessary and edit it. Update all the references where `..\..\..\bin\assembly.dll` to match the apprioprate path. In our example we modified everything to be `..\..\bin\assembly.dll`. When you are done doing this reload your `.csproj` if necessary.
10. Open up your `MSBuild.Community.Tasks.Targets` which is located in the `BuildScripts` folder and modify your relative `MSBuildDnnBinPath` to match your relative path from step 9.
11. Right click on your `.csproj` and navigate to your project properties->Build. Select the `Debug` and `Release` configurations and verify the output path matches your relative path from step 8. 
  1. If you do not do this for both configurations you will run into issues on the CI build
12. Verify that `Debug` build completes successfully
13. Verify that `Release` build completes successfully and generates the `source` and `install` `.zip` files in the `install` folder
14. Push your changes up to your repository we are ready to start configuring the builds.

## Configure CI Build ##
All of our repositories are now created and we are ready to start configuring our Continuous Integration (CI) Build. The CI build will be triggered with each commit and automatically update the version number in the `.dnn` manifest file. After configuring your CI Build you will be able to build your DNN Module indepently of everything else and product the module installer as a build artifact. This module installer can then be ran on any DNN Website and install your module easily.

Let's get started
1. Log into your VSTS instance and navigate to builds
2. Click create a new build and select the empty template
3. Add the following tasks (see detailed info for each task below)
  1. NuGet Restore
  2. Set DNN Version Number
  3. Build Solution
  4. Test Assemblies
  5. Copy Files
  6. Publish Artifacts

#### NuGet ####
1. Add the NuGet restore task
2. You shouldn't have to do anything, but verify your settings
<br />
![CI NuGet Task]({{ site.url }}/assets/posts/2017-10/dnn-vsts-ci-nuget.png)

#### Set DNN Version Number ####
1. Goto the Visual Studio Team Services [Marketplace](https://marketplace.visualstudio.com/vsts)
2. Search for `dnn module set version` or navigate to the [DNN Module Set Version](https://marketplace.visualstudio.com/items?itemName=hoefling-software.SetVersionDNN)
3. Install the VSTS Extension to your project
4. Add the Set DNN Version Number task to the build
5. All the fields should remain the default values
<br />
![CI DNN Version Task]({{ site.url }}/assets/posts/2017-10/dnn-vsts-ci-dnn-version.png)

#### Build Solution ####
1. Add the Build Solution Task

#### Test Assemblies ####
1. Add the Test Assemblies Task

#### Copy Files ####
1. Add the Copy Files Task
2. Modify the regex in Contents to be `**\install\*install.zip` which will prune all files except the installer
3. Update the Target Folder to be `$(build.artifactstagingdirectory)`
4. Check Clean Target Folder
5. Check Flatten Folders
<br />
![CI Copy Files Task]({{ site.url }}/assets/posts/2017-10/dnn-vsts-ci-copy-files.png)

#### Publish Artifacts ####
1. Add the Publish Artifacts Task
2. Modify Path to Publish to `$(build.artifactstagingdirectory)`
3. Modify Artifact Name to `Drop`
<br />
![CI Publish Artifacts Task]({{ site.url }}/assets/posts/2017-10/dnn-vsts-ci-publish-artifacts.png)

When we are all done our build template should look something like this
<br />
![CI Build Tasks]({{ site.url }}/assets/posts/2017-10/dnn-vsts-ci-overview.png)

### Configure CI Build Settings ###
Now that the Build Tasks are all configured correctly we can go through the remaining steps of configuring the build settings.

#### Version Number ####
Navigate to the Build Variables tab and add the following properties and set them to the approriate values you want, for our demo they are both 0

* majorVersion
* minorVersion
<br />
![CI Build Variable]({{ site.url }}/assets/posts/2017-10/dnn-vsts-ci-variables.png)
<br />

Navigate to the Build Options tab and update the property Build Number Format to be in the following format:

* `$(majorVersion).$(minorVersion)$(rev:.r)` - `$(rev:.r)` will specify the latest version of the `minorVersion`

![CI Build Options]({{ site.url }}/assets/posts/2017-10/dnn-vsts-ci-options.png)

#### CI Build Trigger ####
Navigate to the Build Trigger tab and configure your trigger to queue a new build when a commit happens on your master branch.

#### Test CI Build ####
Your CI build should be working at this point, go ahead and give a test. Once you have a successful build verify you are getting your intaller file created as the only build artifact. 
<br />
![CI Build Artifacts]({{ site.url }}/assets/posts/2017-10/vsts-dnn-ci-artifacts.png)

## Configure Resources in Azure ##

## Configure Release Build ##