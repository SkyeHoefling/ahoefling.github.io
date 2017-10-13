---
layout: post
title:  VSTS (Visual Studio Team Services) web.config connection string transforms
modified: 2017-10-13 08:00:00
categories: DevOps
tags: [VSTS, Build, Release, Transform, Azure Deploy]
share: true
comments: true
---
The Release build in VSTS is a powerful tool and once customized correctly can eliminate the need for simple transform files being included in source control. After reading this article you should have an understanding of how to update a web.config for different VSTS Release Build Enviornments.

When you have a full build suite which includes deployment builds you will most likely have a requirment of Xml Transforms. The simple example here is updating you `web.config` to point to different databases for the different environments. 

Here is a list of different environments you may have:

* Dev
* QA
* Staging
* UAT
* Production

I have built complicated transforms that have been included in source control to generate the correct data. This is a great way to solve the problem but there is another way that may meet your needs. I can simple tweak a few settings on my Release Build inside of VSTS and have the transform applied with writing 0 lines of code.

You will need to have a working Release Build and Environment configured before you continue. (Not documented here)

## 1. Find Our Element to Transform ##
The first thing that needs to be done is figure out what needs to be transformed and what the new value if. In our example here we will be transforming the connection string in our `web.config` for a asp.net website.

{% highlight xml linenos %}
<configuration>
  <connectionStrings>
    <add name="DbConnection" connectionString="Data Source=localhost;Initial Catalog=dev;Integrated Security=true;" providerName="System.Data.SqlClient"/>>
  </connectionStrings>
...
</configuration>
{% endhighlight %}

Our variable name here is `DbConnection` this is important to remember when we get to the next step in configuring the transform.

Our connection string in this case points to our local host database and uses the dev database. This is works for local development but our release build will need to use the proper connection string.

## 2. Add Transform Variable to Release Build ##
Now that we have identified our transform variable as `DbConnection` from the previous step it is time to configure our Release Build Variable to perform the transform on deployment.

Open up the Release Build in your VSTS instance, the default view is the Pipeline which is the first tab. You should see several tabs that you can navigate to. Head over to the variables tab and add in the new connection string variable

Variable Format:

* Name - Value - Environment

In my case I have a Staging and a UAT (User Acceptance Testing) server and here is a screenshot of how my variables are configured:

![Release Variables]({{ site.url }}/assets/posts/2017-10/transform-variable.png)

Remember earlier we said we need to remember the connection string name `DbConnection` that is important here and as you can see from my screenshot it matches. The system will automatically find the correct XML element and update it.

## 3. Configure Build Task ##
Now that our Release Build variables have been correctly configured we can configure our build task to enable the XML Transforms.

On the edit page of our VSTS Release Build select the Tasks tab and pick the environment. Another way to get to the correct page is select the pipeline view and navigate to the correct environment in your build process. You will need to do this for each environment that you want to perform the XML Transform on. 

1. Now viewing your tasks you should have an Azure App Service Deploy task, select that task. 
2. Scroll down towards the bottom of the task and expand the "File Transforms & Variable Substitions Options"
3. Once you expand the options there will be several checkboxes, make sure you check the setting "XML Variable Substition"

Here is a screenshot of how my configuration looks:
![XML Variable Substition Setting]({{ site.url }}/assets/posts/2017-10/xml-variable-substition-task.png)

## 4. Deploy ##
Everything is configured correctly and you are ready to deploy. Fire off your build and once it completes everything should be transformed. 

If you would like to manually verify this I would recommend using the Kudu Tools built into Azure and manually verify the connection string is correct.

