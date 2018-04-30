---
layout: post
title:  DNN MVC Module Routing
modified: 2018-04-30 08:00:00
categories: DNN
tags: [DNN, MVC, Module, Routing]
share: true
comments: true
---
DNN 9.2 introduces many new features including new routing controls for MVC Modules. Now, when building a MVC Module you can easily Redirect routes between Controllers and Actions at the Controller level. This new feature introduces flexibility that adds feature parity with Microsoft's MVC implementation. With this change a MVC Module can contain many controllers and actions per controller that handle the complex routing scenarios associated with MVC development. Prior to 9.2 developers were limited to having one controller. While there were workarounds to this limitation until now there wasn't an elegant way to handle routing in a MVC Module.

## What Changed? ##
The `DnnController` is the heart of the DNN MVC Module pattern with several familiar methods and properties overidden for DNN specific purposes. Yet, this was incomplete because there was no `DnnUrlHelper` on the `DnnController`.

### Pull Request 1925 - DnnUrlHelper ###

* [Pull Request 1925](https://github.com/dnnsoftware/Dnn.Platform/pull/1925/)

Adding the `DnnUrlHelper` to the `DnnController` allows us to easily route between Actions and different controllers just like you would in Microsoft's MVC implementation

### Pull Request 1913 - ActionFilter ###

* [Pull Request 1931](https://github.com/dnnsoftware/Dnn.Platform/pull/1931)

The DNN MVC Action Pipeline was updated to handle redirects in the context of an `ActionFilter`. This gives developers power to handling routing scenarios from an `ActionFilter`

## Action Routing ##
Let's take a look at a simple routing problem. Suppose you have built a custom module that has a form where you expect user input. When the user submits the form you want to process that data and redirect to a confirmation screen to let the user know their form submission was successful. 

We will have a `DnnController` with the following actions:

| Method | Action       |
|--------|--------------|
| GET    | Index        |
| POST   | Index        |
| GET    | Confirmation |

<br />

{% highlight c# linenos %}
public class HomeController : DnnController
{
    public ActionResult Index()
    {
        return View();
    }

    [HttpPost]
    public ActionResult Index(FormInput data)
    {
        return Redirect(Url.Action("Confirmation", "Home"));
    }

    public ActionResult Confirmation()
    {
        return View("Confirmation");
    }
}
{% endhighlight %}

The glue to this code is in our `HttpPost` where we redirect to the Confirmation Action

* `Redirect(Url.Action("Confirmation", "Home"))`

Breaking down this statement there are 2 big pieces

* `Redirect`
* `Url.Action`

`Redirect` is an ASP.NET method that forces a redirect to a specific URL provided as the parameter

`Url.Action` is using the `DnnUrlHelper` to generate the DNN appropriate URL to pass into the `Redirect` method

## Controller Routing ##
Now that we understand a simple routing scenario let's expand our problem to solve a more complex scenario. Building on our original problem of a simple form that helps us collect user data. Suppose the form is collecting public data that anyone can view and after the user submits their form they can view a list of submissions by other users.

We are now describing a much more complicated module let's describe each page in a little more detail

| Page          | Description                                          |
|---------------|------------------------------------------------------|
| Form Input    | A simple form that takes user input and submits to the controller. |
| Confirmation  | A confirmation screen that tells the user their data has been submitted. This screen has a button to navigate to the result list to see all submissions in the system. |
| Result List   | A list of all form submissions organized by date. When the user clicks on an item it navigates to the Input Details screen. |
| Input Details | Details of a previously submitted form item. |

<br />

Let's describe our different Controllers and Actions

| Controller     | Action   | Method       |
|----------------|----------|--------------|
| HomeController | GET      | Index        |
| HomeController | POST     | Index        |
| HomeController | GET      | Confirmation |
| ListController | GET      | Index        |
| ListController | GET      | Details      |

<br />

We have 2 Controllers and multiple Actions that have different interactions. Now that we have defined what we are trying to build, let's create our controllers

HomeController
{% highlight c# linenos %}
public class HomeController : DnnController
{
    public ActionResult Index()
    {
        return View();
    }

    [HttpPost]
    public ActionResult Index(FormInput data)
    {
        return Redirect(Url.Action("Confirmation", "Home"));
    }

    public ActionResult Confirmation()
    {
        return View("Confirmation");
    }
}
{% endhighlight %}  

The HomeController is IDENTICAL to what we did earlier, nothing changes. The change that you will need to implement is routing an anchor on your view. So you should add the following code somewhere on you `Confirmation.cshtml`

{% highlight html linenos %}
<a href="@Url.Action("Index", "List")">View All Results</a>
{% endhighlight %}

{% highlight c# linenos %}
public class ListController : DnnController
{
    public ActionResult Index()
    {
        return View();
    }

    public ActionResult Details(int inputId)
    {
        // retrieve your input here and pass your model into the view
        return View("Details");
    }
}
{% endhighlight %}  

To breakdown what we are doing here:

Utilizing the `DnnUrlHelper` to build an apprioprate DNN url and using the `Controller` method `Redirect` to navigate to the new route. In our case here we are utilizing the same logic (ex: `Url.Action("Index", "Home")`) in both the Controller and the View where applicable.

## Filter Routing ##
At this point you should have a MVC Module that has mutliple controllers and handles routing at both the controller level and the view level where applicable. This is a fantastic improvement over how routing used to be handled in MVC Modules. Let's say our module throws an exception while it is running. It would be really useful to create an `ActionFilter` that intercepts the exception and processes a redirect. This will allow us to display a user friendly screen to the end-user of the module.

Let's create an `ActionFilter` for exception handling, which is exactly how you would handle this scenario in ASP.NET MVC

{% highlight c# linenos %}
public class RedirectOnExceptionAttribute : ActionFilterAttribute, IExceptionFilter
{
    public void OnException(ExceptionContext filterContext)
    {
        var controller = (HomeController)filterContext.Controller
        filterContext.Result = controller.RedirectToError();
        filterContext.ExceptionHandled = true;
    }
}
{% endhighlight %}

Looking at the code above we need to define a special controller method to handle the redirect, let's define that as well

{% highlight c# linenos %}
public class HomeController : DnnController
{
    // omitted other methods

    public ActionResult Error()
    {
        return View();
    }

    public ActionResult RedirectToError()
    {
        return Redirect(Url.Action("Error", "Home"));
    }
}
{% endhighlight %}

There are a couple big pieces to the `ActionFilter` Scenario, let's break them down

* The `ActionFilter` to catch the exception
* Setting the `filterContext.Result` which is of type `ActionResult`
* Configuring your `DnnController` to have a method and/or Action available to handle the error redirect

Your `ActionFilter` code should be almost identical to how you would solve this same problem in an ASP.NET MVC website. The trick is having your `HomeController` in our example configured to return an `ActionResult` that can be processed.

Note that the `HomeController` now has 2 Actions 

* Error
* RedirectToError

The `ActionResult` that is generated from these methods is pipped into the `filterContext` and when the `ActionFilter` finishes processing the page will be updated.

My personal preference is setting up these 2 Actions. This allows us to specify a Redirect which will update the url in the browser and completely move the user off the page that errored out. You technically don't have to use the 2 actions but I consider it a best practice for handling the routes.