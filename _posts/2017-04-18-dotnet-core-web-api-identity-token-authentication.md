---
layout: post
title:  Dotnet Core Web API with Identity Token Authentication
modified: 2017-04-18 08:00:00
categories: dotnet core
tags: [dotnet core, Web API, asp.net, Identity, EF Core, Token, Authentication, jQuery]
share: true
comments: true
---
When you set out to create a new web application in ASP.NET you have 2 major choices: 
* MVC
* Web API

Today we are going to take a look at creating necessary APIs for user authentication. 

* ASP.NET Core Identity
 * Authentication
 * Saving Cookies
 * Generating Tokens
* Create Scaffolding for Web API

## Follow the MVC Tutorial ##
There is a fantastic tutorial to setting up ASP.NET MVC Core with Identity Authentication that generates tokens that are then stored in the browsers cookies. The author of the tutorial wrote this specifically for MVC Razor and when I went through it I decided to implement a solution using Web API. 

* [http://www.blinkingcaret.com/2016/11/30/asp-net-identity-core-from-scratch/](http://www.blinkingcaret.com/2016/11/30/asp-net-identity-core-from-scratch/)
* Rui Figueiredo

If you are not familiar with the ASP.NET MVC/Web API Architecture I would recommend following the tutorial to give you a beter idea. If not skip ahead and dive right in.

### Development Environment ###
* [Ubuntu 16.04](https://www.ubuntu.com/)
* [dotnet core 1.0.1](https://www.microsoft.com/net/core#linuxubuntu)
* [VS Code 1.11.2](https://code.visualstudio.com/)

## Modified Tutorial ##
This is a modified tutorial that follows the same methods and teachings as the blog mentioned above. Everything here posted below is my own adaptations from the original blog. I would like to give credit to the original post for showing me everything I needed to do to get this working correctly.

Let's get started by running a few commands in your terminal

{% highlight xml linenos %}
dev@machine:~$ mkdir tutorial
dev@machine:~$ cd ./tutorial
dev@machine:/tutorial$ dotnet new webapi
{% endhighlight %}

You will now have a new Web API project located in `~/tutorial` which will generate all the scaffolding needed. 

At this point I like to see if everything is working and confirm that I have my environment configured correctly, inside your terminal run the following commands and make sure they complete without errors

{% highlight xml linenos %}
dev@machine:~/tutorial$ dotnet restore
dev@machine:~/tutorial$ dotnet build
dev@machine:~/tutorial$ dotnet run
{% endhighlight %}

### NuGet Packages ###
Add the following NuGet libraries with the following commands:

{% highlight xml linenos %}
dev@machine:~/tutorial$ dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dev@machine:~/tutorial$ dotnet add package Microsoft.EntityFrameworkCore.Designer
dev@machine:~/tutorial$ dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dev@machine:~/tutorial$ dotnet add package Microsoft.EntityFrameworkCore.Tools.DotNet
{% endhighlight %}

Once you succesfully run all the commands open up your `tutorial.csproj` file which should look something like this:

{% highlight xml linenos %}
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>netcoreapp1.1</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <Folder Include="wwwroot\" />
  </ItemGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore" Version="1.1.1" />
    <PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="1.1.1" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc" Version="1.1.2" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="1.1.1" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="1.1.1" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="1.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="1.1.1" />
  </ItemGroup>
</Project>
{% endhighlight %}

There is a problem with the project file above that we need to correct. Line 13 reads 

* `<PackageReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="1.0.0" />` 

You will need to update it to the following:

* `<DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="1.0.0" />`

### Configure Identity Core Database ###
Open up the `Startup.cs` file and navigate to the `ConfigureServices` method. Modify it to look like the following snippet (we will go over everything in detail in the sub-sections):

`Startup:ConfigureServices`
{% highlight c# linenos %}
public void ConfigureServices(IServiceCollection services)
{
    // configure database
    var server = "localhost"; 
    var connection = $@"Server={server};Database=purchasing_dev;user=sa;password=Password01!;";
    services.AddDbContext<IdentityDbContext>(options => options.UseSqlServer(
        connection,
        optionsBuilder => optionsBuilder.MigrationsAssembly("WebApiIdentityTokenAuth")));

    // configure token generation
    services.AddIdentity<IdentityUser, IdentityRole>()
        .AddEntityFrameworkStores<IdentityDbContext>()
        .AddDefaultTokenProviders();
    
    // configure identity options
    services.Configure<IdentityOptions>(o => {
        o.SignIn.RequireConfirmedEmail = true;
    });
}
{% endhighlight %}

Finally in the `Configure` method we need to specify that we want to use idenity by `app.UseIdentity()`

`Startup:Configure`
{% highlight c# linenos %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    app.UseIdentity();
}
{% endhighlight %}

#### Configure Database Connection ####
To configure the databsae all you need is a user that can create databases. Everything else is built right into the Identity libraries. You don't need to create any special DbContext's or anything. You should be able to just run the following statements and you are good to go.

1. Create connection string
2. Configure `IdentityDbContext` to use connection string

{% highlight c# linenos %}
var server = "localhost"; 
var connection = $@"Server={server};Database=purchasing_dev;user=sa;password=Password01!;";
services.AddDbContext<IdentityDbContext>(options => options.UseSqlServer(
    connection,
    optionsBuilder => optionsBuilder.MigrationsAssembly("api")));
{% endhighlight %}

#### Configure Token Generation ####
This snippet configures our `IdentityUser` to use the Token provider. This is a super important step which helps generate our custom cookie that is sent to the client. Without the cookie the user will have to re-authenticate on each call.

{% highlight c# linenos %}
services.AddIdentity<IdentityUser, IdentityRole>()
    .AddEntityFrameworkStores<IdentityDbContext>()
    .AddDefaultTokenProviders();
{% endhighlight %}

#### Configure Identity Options ####
We may elect certain password security rules, with Identity it is super easy to do this. Our current policy is enabling email confirmation.

{% highlight c# linenos %}
services.Configure<IdentityOptions>(o => {
    o.SignIn.RequireConfirmedEmail = true;
});
{% endhighlight %}



### Enable CORS ###
Since we are using Web API instead of MVC it will be useful to enable CORS which allows us to use our Web API from other sites than the one that the Web API is hosted on.

* `Startup:ConfigureServices`
{% highlight c# linenos %}
services.AddCors()
{% endhighlight %}
* `Startup:Configure`
{% highlight c# linenos %}
app.UseCors(b => b.WithOrigins("http://dev.localhost.com:4000")
    .AllowAnyOrigin()
    .AllowCredentials()
    .AllowAnyMethod()
    .AllowAnyHeader());
{% endhighlight %}

### Dependency Injection ###
We touched on confirming emails earlier when the user creates their account. It will be useful to inject a service into our controllows so let's add some code into the `ConfigureServices` method that does just that.

`Startup: ConfigureServices`
{% highlight c# linenos %}
services.AddTransient<IMessageService, FileMessageService>();
{% endhighlight %}

`IMessageService Interface`
{% highlight c# linenos %}
public interface IMessageService
{
    Task Send(string email, string subject, string message);
}  
{% endhighlight %}

We are not going to implement this interface in the tutorial here, but you can view the code or reference the MVC Identity Post mentioned above. This is here for your reference.

#### Troubleshooting ####
Please provide your own implementation for these classes or the project will not build.

### Create the Database ###
If you have made it this far and everything builds then you are ready to create your database. Open up the terminal and run the following commands:

{% highlight xml linenos %}
dev@machine:~/tutorial$ dotnet ef migrations add InitialIdentityCreate
dev@machine:~/tutorial$ dotnet ef database update
{% endhighlight %}

* `dotnet ef migrations add InitialIdentityCreate`
 * This command will create the necessary migration files that will create your database. This is a scaffolding command
* `dotnet ef database update`
 * This will run your migration scaffolding against the database that is specifeid in your `Startup.cs` file.

#### Troubleshooting ####
If you can't get these commands to work confirm that you have successfully added `Microsoft.EntityFrameworkCore.Tools.DotNet` to your `tutorial.csproj` file.

### Build the API ###
Now that everything is configured let's build out our API! As before we are going to show the entire class to start and then go through each step in detail.

When I am building a Web API I like to explicitly specify all my routes to make it super clear on how to call the method and retrieve that data. You will see the methods appended by `[Route("methodName")]`

`AccountController`
{% highlight c# linenos %}
[Route("api/[controller]")]
public class AccountController : Controller
{  
    private readonly UserManager<IdentityUser> _userManager;
    private readonly SignInManager<IdentityUser> _signInManager;
    private readonly IMessageService _messageService;

    public AccountController(
        UserManager<IdentityUser> userManager,
        SignInManager<IdentityUser> signInManager,
        IMessageService messageService)
    {
        _userManager = userManager;
        _signInManager = signInManager;
        _messageService = messageService;
    }

    [HttpPost]
    [Route("register")]
    public async Task<JsonResult> Register(string email, string password, string confirmPassword)
    {
        if (string.IsNullOrWhiteSpace(email) || string.IsNullOrWhiteSpace(password))
        {
            return Json(new Response(HttpStatusCode.BadRequest)
            {
                Message = "email or password is null"
            });
        }

        if (password != confirmPassword)
        {
            return Json(new Response(HttpStatusCode.BadRequest)
            {
                Message = "Passwords don't match!"
            });
        }

        var newUser = new IdentityUser
        {
            UserName = email,
            Email = email
        };

        IdentityResult userCreationResult = null;
        try
        {
            userCreationResult = await _userManager.CreateAsync(newUser, password);
        }
        catch(SqlException)
        {
            return Json(new Response(HttpStatusCode.InternalServerError)
            {
                Message = "Error communicating with the database, see logs for more details"
            });
        }

        if (!userCreationResult.Succeeded)
        {
            return Json(new Response(HttpStatusCode.BadRequest)
            {
                Message = "An error occurred when creating the user, see nested errors",
                Errors = userCreationResult.Errors.Select(x => new Response(HttpStatusCode.BadRequest)
                {
                    Message = $"[{x.Code}] {x.Description}"
                })
            });
        }

        var emailConfirmationToken = await _userManager.GenerateEmailConfirmationTokenAsync(newUser);
        var tokenVerificationUrl = Url.Action(
            "VerifyEmail", "Account",
            new {
                Id = newUser.Id,
                token = emailConfirmationToken
            },
            Request.Scheme);
        
        await _messageService.Send(email, "Verify your email", $"Click <a href=\"{tokenVerificationUrl}\">here</a> to verify your email");
        
        return Json(new Response(HttpStatusCode.OK){
            Message = $"Registration completed, please verify your email - {email}"
        });
    }

     public async Task<IActionResult> VerifyEmail(string id, string token)
    {
        var user = await _userManager.FindByIdAsync(id);
        if (user == null)
            throw new InvalidOperationException();

        var emailConfirmationResult = await _userManager.ConfirmEmailAsync(user, token);
        if (!emailConfirmationResult.Succeeded)
        {
            return new RedirectResult("http://dev.localhost.com:4000/registration.html");
        }

        return new RedirectResult("http://dev.localhost.com:4000/");
    }  

    [HttpPost]
    [Route("login")]
    public async Task<JsonResult> Login(string email, string password)
    {
        if (string.IsNullOrWhiteSpace(email) || string.IsNullOrWhiteSpace(password))
        {
            return Json(new Response(HttpStatusCode.BadRequest)
            {
                Message = "email or password is null"
            });
        }

        var user = await _userManager.FindByEmailAsync(email);
        if (user == null)
        {
            return Json(new Response(HttpStatusCode.BadRequest)
            {
                Message = "Invalid Login and/or password"                    
            });
        }

        if (!user.EmailConfirmed)
        {
            return Json(new Response(HttpStatusCode.BadRequest)
            {
                Message = "Email not confirmed, please check your email for confirmation link"
            });
        }

        var passwordSignInResult = await _signInManager.PasswordSignInAsync(user, password, isPersistent: true, lockoutOnFailure: false);
        if (!passwordSignInResult.Succeeded)
        {
            return Json(new Response(HttpStatusCode.BadRequest)
            {
                Message = "Invalid Login and/or password"
            });
        }

        return Json(new Response(HttpStatusCode.OK)
        {
            Message = "Cookie created"
        });
    }

    [HttpPost]
    [Route("logout")]
    public async Task<JsonResult> Logout()
    {
        await _signInManager.SignOutAsync();

        return Json(new Response(HttpStatusCode.OK)
        {
            Message = "You have been successfully logged out"
        });
    }
}
{% endhighlight %}

#### Inject All the Dependencies ####
Let's inject everything that we are going to need. When we went through the configuration phase of the tutorial we registered a bunch of dependencies that we are going to be using in the `AccountController`

{% highlight c# linenos %}
private readonly UserManager<IdentityUser> _userManager;
private readonly SignInManager<IdentityUser> _signInManager;
private readonly IMessageService _messageService;

public AccountController(
    UserManager<IdentityUser> userManager,
    SignInManager<IdentityUser> signInManager,
    IMessageService messageService)
{
    _userManager = userManager;
    _signInManager = signInManager;
    _messageService = messageService;
}
{% endhighlight %}

#### Register a New User ####
The following steps produce the registration process

1. Verify we have an email and password
2. Confirm the passwords match
3. Create the `IdentityUser` and attempt to save it
4. Check the results object for errors and if they exist return
5. Generate an email confirmation token and send it to the provided email

{% highlight c# linenos %}
[HttpPost]
[Route("register")]
public async Task<JsonResult> Register(string email, string password, string confirmPassword)
{
    if (string.IsNullOrWhiteSpace(email) || string.IsNullOrWhiteSpace(password))
    {
        return Json(new Response(HttpStatusCode.BadRequest)
        {
            Message = "email or password is null"
        });
    }

    if (password != confirmPassword)
    {
        return Json(new Response(HttpStatusCode.BadRequest)
        {
            Message = "Passwords don't match!"
        });
    }

    var newUser = new IdentityUser
    {
        UserName = email,
        Email = email
    };

    IdentityResult userCreationResult = null;
    try
    {
        userCreationResult = await _userManager.CreateAsync(newUser, password);
    }
    catch(SqlException)
    {
        return new Response(HttpStatusCode.InternalServerError)
        {
            Message = "Error communicating with the database, see logs for more details"
        };
    }

    if (!userCreationResult.Succeeded)
    {
        return new Response(HttpStatusCode.BadRequest)
        {
            Message = "An error occurred when creating the user, see nested errors",
            Errors = userCreationResult.Errors.Select(x => new Response(HttpStatusCode.BadRequest)
            {
                Message = $"[{x.Code}] {x.Description}"
            })
        };
    }

    var emailConfirmationToken = await _userManager.GenerateEmailConfirmationTokenAsync(newUser);
    var tokenVerificationUrl = Url.Action(
        "VerifyEmail", "Account",
        new {
            Id = newUser.Id,
            token = emailConfirmationToken
        },
        Request.Scheme);
    
    await _messageService.Send(email, "Verify your email", $"Click <a href=\"{tokenVerificationUrl}\">here</a> to verify your email");
    
    return Json(new Response(HttpStatusCode.OK){
        Message = $"Registration completed, please verify your email - {email}"
    });
}
{% endhighlight %}

#### Verify a New User ####
When we verify the new user we redirect them either to the login page or the registration page depending on if it is a valid verificaiton token or not.

{% highlight c# linenos %}
public async Task<IActionResult> VerifyEmail(string id, string token)
{
    var user = await _userManager.FindByIdAsync(id);
    if (user == null)
        throw new InvalidOperationException();

    var emailConfirmationResult = await _userManager.ConfirmEmailAsync(user, token);
    if (!emailConfirmationResult.Succeeded)
    {
        return new RedirectResult("http://dev.localhost.com:4000/registration.html");
    }

    return new RedirectResult("http://dev.localhost.com:4000/");
}  
{% endhighlight %}

#### Login ####
The login method is pretty straight forward, we take in the email and password and verify they are a user. Then we call the Identity APIs verifying they are correct which will return an authentication cookie which gets stored as a cookie on the browser.

It is important to note that we aren't doing anything special to store the cookie this is handled automatically by the Web API and the Identity Library.

{% highlight c# linenos %}
[HttpPost]
[Route("login")]
public async Task<JsonResult> Login(string email, string password)
{
    if (string.IsNullOrWhiteSpace(email) || string.IsNullOrWhiteSpace(password))
    {
        return Json(new Response(HttpStatusCode.BadRequest)
        {
            Message = "email or password is null"
        });
    }

    var user = await _userManager.FindByEmailAsync(email);
    if (user == null)
    {
        return Json(new Response(HttpStatusCode.BadRequest)
        {
            Message = "Invalid Login and/or password"                    
        });
    }

    if (!user.EmailConfirmed)
    {
        return Json(new Response(HttpStatusCode.BadRequest)
        {
            Message = "Email not confirmed, please check your email for confirmation link"
        });
    }

    var passwordSignInResult = await _signInManager.PasswordSignInAsync(user, password, isPersistent: true, lockoutOnFailure: false);
    if (!passwordSignInResult.Succeeded)
    {
        return Json(new Response(HttpStatusCode.BadRequest)
        {
            Message = "Invalid Login and/or password"
        });
    }

    return Json(new Response(HttpStatusCode.OK)
    {
        Message = "Cookie created"
    });
}
{% endhighlight %}

#### Logout ####
The Logout logic is super easy with the Identity library, just call `SignOutAsync()` and we are done!

{% highlight c# linenos %}
[HttpPost]
[Route("logout")]
public async Task<JsonResult> Logout()
{
    await _signInManager.SignOutAsync();

    return Json(new Response(HttpStatusCode.OK)
    {
        Message = "You have been successfully logged out"
    });
}
{% endhighlight %}

### Test the API ###
At this point I would recommend using a tool to test your API that is able to receive cookies. Below is a screenshot of what the cookie looks like in postman

![Postman Cookie]({{ site.url }}/assets/posts/2017-04/postman-cookie.png)

### Front-End - Javascript with jQuery ###
There are countless options of how to develop a front end from simple jQuery to Angular to a mobile app. We aren't going to go through every possible front end option. That is the whole point of creating a Web API being able to swap out the front end with a different implementation. 

What we are going to do is talk about jQuery and how to use it with your API. When I went through this on my own I spent several hours trying to figure out what I was doing wrong. 

This example may not be the best way to handle your UI, but it is a quick and dirty way to demonstrate using the Web API

`Login`
{% highlight javascript linenos %}
$.ajax({
    type: 'post',
    url: 'http://dev.localhost.com:5000/v1/api/account/login',
    data: user,
    xhrFields: {
        withCredentials: true
    }
}).done(function(data){
    if(data.code == 200)
    {
        window.location.href = "http://dev.localhost.com:4000/home.html";
    } 
    else
    {
        account.$loginErrorCallout.html(data.message);
        account.$loginErrorCallout.show();
    }
});
{% endhighlight %}

#### Credential Passing ####
You may be asking yourself, because I did, "Why can't I just use `$.post()`, what is so special about `$.ajax()`". If you do not specify `xhrFields.withCredentials: true` none of this will work.

The cookie will be returned like the Web API always does from the `login` method but it wont' be saved. All the methods from here on out that use the cookie will need be setup passing the proper `xhrFields.withCredentials` property. 

## GitHub Example ##
You did it!! At this point you should have a working Web API with Identity Token Authentication. If you are having trouble just fork my GitHub sample that has all the Web API code from this tutorial.

[http://www.github.com/ahoefling/WebApiIdentityTokenAuth](http://www.github.com/ahoefling/WebApiIdentityTokenAuth)
