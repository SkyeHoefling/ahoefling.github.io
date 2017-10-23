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
Before we can start deploying resources you need to add some resources into your Azure Subscriptions. Please go ahead and create the following resources to mimic our enviornment in your subscription

| Resource             |
|----------------------|
| Web App              |
| SQL Database Server  |

<br />
If you would like to to create a deployment for each environment it may be useful to create this table instead

| Resource                      | Options                        |
|-------------------------------|--------------------------------|
| Web App - Dev                 | N/A                            | 
| Web App - Staging             | N/A                            |
| Web App - UAT                 | N/A                            |
| Web App - Production          | N/A                            |
| SQL Database Server           | 4 Database, 1 for each web app |

<br />

## Configure Release Build ##
All the Azure resources are configured and we are ready to configure the Release Builds which will deploy the new website and auto install the modules in our Azure Resources. 

![DNN Release Build Workflow]({{ site.url }}/assets/posts/2017-10/DnnCodeMergeAndCleanDeployment.png)

### Configure Artifacts ###
Once you have clicked the Add New Release button you are brought to the pipeline screen, we need to start adding our artifacts which will all be merged together.

#### DNN Module ####
1. In the Artifacts section click the `+ Add` button to add your artifact
2. Add the DNN Module CI Build Artifacts
<br />
![DNN Release Artifact Module]({{ site.url }}/assets/posts/2017-10/dnn-vsts-release-artifact-module-build.png)

#### DNN Website ####
1. In the Artifacts section click the `+ Add` button to add your artifact
2. Add the DNN Website Repository
<br />
![DNN Release Artifact Module]({{ site.url }}/assets/posts/2017-10/dnn-vsts-release-artifact-website-build.png)

### Create Enviornment ###
1. In the Environments section click the `+ Add` button to create our new enviornment
2. Select new environment
3. You will be brought to select build template screen where you want to select an empty template
4. Now give your enviornment a name
5. Create Release Build Tasks
    1. Stop Azure App Service
    2. Copy Website Artifacts to Merged Directory
    3. Copy Module Artifacts to Merged Directory
        1. Repeat for each module
    4. Deploy Azure App Service
    5. Start Azure App Service
    6. Azure Powershell CLI - Drop and Create Databases

#### Stop Azure App Service ####
Task - Azure App Service Manage

1. Select your Azure Subscription
2. Select the action `Stop App Service`
3. Select your App Service Name

#### Copy Website Artifacts ####
Task - Copy Files

1. Specify your source folder which you can select with the `...`.
2. Specify your target folder which we chose `$(System.DefaultWorkingDirectory)/merged`
<br />
![DNN Release Task Copy Website]({{ site.url }}/assets/posts/2017-10/dnn-vsts-release-env-task-copy-website.png)

#### Copy Module Artifacts ####
Task - Copy Files

1. Specify your build artifacts drop location via the `...`
2. Specify your target folder making sure to drop it into the `install/module` directory
    * `$(System.DefaultWorkingDirectory)/merged/Install/Module`
<br />
![DNN Release Task Copy Module]({{ site.url }}/assets/posts/2017-10/dnn-vsts-release-env-task-copy-module.png)

#### Deploy Azure App Service ####
Task - Azure App Service Deploy

1. Select your Azure Subscription
2. Select your Azure App Service Name
3. Select the folder specified from the previous copy steps
    * `$(System.DefaultWorkingDirectory)/merged`
4. Under File Transforms & Variable Substitution Options check `XML variable substitution`
    * This is needed to update the database connection string

#### Start Azure App Service ####
Task - Azure App Service Manage

1. Select your Azure Subscription
2. Specify the Action `Start App Service`
3. Select your App Service Name

#### Drop and Create Database ####
Task - Azure PowerShell

1. Select Azure Connection Type of Azure Resource Manager
2. Select Azure Subscription
3. Select Script type of inline script
4. Include the following script where you update the following parameters to match your own

Update the following parameters
| Parameter           | Description                                                             |
|---------------------|-------------------------------------------------------------------------|
| ResourceGroupName   | The Azure Resource where your app and database server exist             |
| ServerName          | The Database Server name                                                |
| DatabaseName        | The SQL Database name that is contained on the Database Server          |

{% highlight bash linenos %}
# You can write your azure powershell scripts inline here. 
# You can also pass predefined and custom variables to this script using arguments
Remove-AzureRMSqlDatabase -ResourceGroupName "DNNDemo" -ServerName "dnn-demo" -DatabaseName "dnn-demo-dev"
New-AzureRMSqlDatabase -ResourceGroupName "DNNDemo" -ServerName "dnn-demo" -DatabaseName "dnn-demo-dev" -RequestedServiceObjectiveName "Basic"
{% endhighlight %}

#### Completed Workflow ####
![DNN Release Env Tasks]({{ site.url}}/assets/posts/2017-10/dnn-vsts-release-env-tasks.png)

#### Configure Environment Variables ####
Now that the environment is all set up we need to specify the connection string variables so it can be properly transformed at build time.

1. Select the Variables tab
2. Click the `+ Add` button to create a new variable
3. Specify the name of `SiteSqlServer` which is the database connection string
4. Specify the connection string in the Value column
    * The connection string can be retrieved from your Azure Configuration
5. Select the scope which references your enviornment
    * Azure Dev
    * Azure Staging
    * Azure Production
<br />
![DNN Release Env Variable Transforms]({{ site.url }}/assets/posts/2017-10/dnn-vsts-release-env-variable-transforms.png)


### Configure Trigger ###
Configure the trigger by clicking the trigger button on the artifact you would like to trigger. The trigger icon is the lightning bolt icon. See highlighted icon in image below
<br />
![DNN Release Artifact Trigger Icon]({{ site.url }}/assets/posts/2017-10/dnn-vsts-release-artifact-trigger.png)
<br />
1. Module - After selecting the Artifact trigger, just make sure it is enabled and there isn't really anthing else you need to define for the build. 
2. Website - After selecting the artifact trigger make sure you select your clean branch so it knows what branch code to deploy

Now click the trigger icon from the Dev Environment so we can setup the environment to queue when the CI Build completes.
<br />
![DNN Release Env Trigger]({{ site.url }}/assets/posts/2017-10/dnn-vsts-release-env-trigger.png)

### Test Release Build ###
At this point your release build is all set and you should be able to successfully deploy your release build to your Azure Resources. Go ahead and test this to make sure it is working.

When you navigate to your website you should see the install screen. Currently this process does not include auto installs. I plan to build a VSTS Extension in the future to handle this. Currently you manually install it or kick of the web request install process

* http://mywebsite.com/Install.aspx?mode=install

## Other Resources ##
If you have followed everything here correctly when you check in changes to your DNN Module it will now trigger the following actions

| Build | Output |
|--------------------------|------------------------------------------------------------|
| Continuous Integartation | DNN `.zip` Module Installer                                |
| Continuous Deployment    | Clean install of DNN with specified Module Installers      |

<br />
Here are additional resources that may be useful on this topic

* [Powerpoint Presentation Slids](http://bit.ly/2ipPmpm)
* [Southern Fried DNN Presentation](http://southernfrieddnn.com/Blog/TabId/208/PostId/49/october-sofried-meeting-dnn-continuous-delivery-with-vsts-and-azure.aspx)
    * Contains video walkthrough