---
layout: post
title:  How to Setup Swagger in Web API ASP.NET with Swashbuckle
modified: 2017-10-01 08:00:00
categories: dotnet
tags: [dotnet, asp.net, swagger, documentation]
share: true
comments: true
---
When I first tried using Swagger with Web API I spent a little time looking through the configuration files and it seemed a little confusing to me. I did a little research and I came across Swashbuckle which makes adding Swagger UI into your asp.net Web API project a no brainer. It can hook right into your Startup Configuration class and should only take a few minutes to get the basics up and running. After learning how easy it is to setup Swagger UI in my Web API project I now spend the 5 minutes to add it in.

Today we are going to go over the basic features to get going with Swashbuckle:

* Install Swashbuckle
* Overview of configuration code
* How to use API Keys

## Install Swashbuckle ##
Swashbuckle is an open source repository built for both asp.net and asp.net core

GitHub Repos: 

* Swashbuckle - [https://github.com/domaindrivendev/Swashbuckle](https://github.com/domaindrivendev/Swashbuckle)
* Swashbuckle.AspNetCore - [https://github.com/domaindrivendev/Swashbuckle.AspNetCore](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)

NuGet Repos:

* Swashbuckle - [https://www.nuget.org/packages/Swashbuckle](https://www.nuget.org/packages/Swashbuckle)
* Swashbuckle.AspNetCore - [https://www.nuget.org/packages/Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore)

Don't worry about downloading the github repo above, it is here just for your reference so you can take a look at the code or documentation if needed. One thing I really like about this library is how much control you have.

To get started simple add your NuGet Dependency like you normally would. Since are working with asp.net Web API we are going to use the .NET Framework version. 

In the Package Manager console enter the following command
{% highlight powershell linenos %}
Package-Install Swashbuckle
{% endhighlight %}

## Configuration Code ##

You should now have a `SwaggerConfig.cs` file located in you `App_Start` folder. Since we are using Owin Configuration there is now a simple command that needs to be called `SwaggerConfig.Register(config);`. This will call the necessary configuration code that should be created for you.

{% highlight C# linenos %}
public class Startup
{
    public void Configuration(IAppBuilder application)
    {
        var config = new HttpConfiguration();
        config.MapHttpAttributeRoutes();
        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "v1/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional });

        // This is the call to our swashbuckle config that needs to be called
        SwaggerConfig.Register(config);

        // invoke the web API
        application.UseWebApi(config);
    }
}
{% endhighlight %}

### Swagger Config ###
The nice thing when you install the library is it puts a lot of the documentation in the code scaffolding so you don't need to go hunting for it. By default the code scaffolding has most of the optional features commented out with docs right in the comments above each one and how it should be used. 

You can choose to keep this verbose configuration file or shrink it down. I prefer to keep the verbose comments in because I find it useful for me or other devs on the team when we need to trun new features on for the swagger config. Our code snippet below is trimmed down for our purposes here, but the code sample attached is not.

{% highlight C# linenos %}
public class SwaggerConfig
{
    public static void Register(HttpConfiguration config)
    {
        config
            .EnableSwagger()
            .EnableSwaggerUi();
    }
}
{% endhighlight %}

Both `EnableSwagger` and `EnableSwaggerUi` take configuration options which is where you will see all the documentation provided. To get the barebones install running you just need to call the 2 commands above.

### API Keys ###
A common configuration with swagger is enabling API Keys to handle authorization to the API. When you are using a tool such as Postman you may include an API Key in the header. We have this same control with Swagger. There is a input control at the top of the page asking for an API Key. The user just plugs in their key and hits the `Explore` button. Now when they try to use any of the APIs the API Key is sent in the header and the page is once again usable.

How do you configure this? Going back to our configuration file `SwaggerConfig.cs` you need to specify the API Key in both methods `EnableSwagger` and `EnableSwaggerUi`

{% highlight C# linenos %}
public class SwaggerConfig
{
    public static void Register(HttpConfiguration config)
    {
        config
            .EnableSwagger(c =>
            {
                c.ApiKey("apiKey")
                    .Description("API Key for accessing secure APIs")
                    .Name("Api-Key")
                    .In("header");
            })
            .EnableSwaggerUi(c =>
            {
                c.EnableApiKeySupport("Api-Key", "header");
            });
    }
}
{% endhighlight %}

The most important parameters are the `Name()` and `In()` where the API Key name is specified in Name and `In()` determines where to place the API Key. In our example here we want this place in the header and we want it to be called "API-Key".

After we specify the ApiKey in the first configuration section we need to tell SwaggerUi to enable the API Key which we again specify the Name and In parameters again.

At this point if you reload your swagger UI you will be able to specify your API Key and call your Web API through the Web Utility

## Access Swagger UI from Browser ##
We have gone over the basics of setting up swagger UI but did not go over how to access our new API that is apart of our page.

* http://localhost/swagger/ui

## Code Sample ##

TODO Add GitHub repository