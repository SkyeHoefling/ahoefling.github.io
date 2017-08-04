---
layout: post
title:  Intro to Azure Mobile App Services
modified: 2017-04-18 08:00:00
categories: Xamarin
tags: [Xamarin, Azure, App Serivce, OData]
share: true
comments: true
---
The single biggest problem I have seen while developing any type of mobile app is how do we handle offline sync? On mosst projects I have worked on this has been punted as a problem that isn't worth the devs time until we are close to release. While this is a bad idea the team does not need to freak out about handling offline sync. It is easier than we make it for ourselves. Today's take away is "Don't freak out, mobile sync is easy"

Most mobile apps require some backend API or Service Layer that needs to be used to retrieve data. This is true for a wide range of apps from enterprise apps to games and touches upon almost everything in between. This service layer is critical for getting information to the users. Some examples may be.

* OData
* RESTful Service (Web API)

## The Problem and Solution ##
When a mobile app is online querying against this service layer it is trival and there is no need worry about how we are going to get our data to the user. When the user enters offline mode for whatever reason this is where things get very tricky very quickly, but as we said earlier "Don't freak out". Let's first talk about the problem with our current infrastructure, what problems it presents and how we are going to fix it.

<strong>Consider the following architecture diagram:<strong>
![simple mobile architecture]({{ site.url }}/assets/posts/2017-09/simple-mobile-app-architecture.png)

Impotant items to note

* Each app code uses the API and Local Database
* The app code handles doing data diff between the local database and the API

As an app developer I do <strong>not</strong> want to spend my time writing code that merges data from 1 data source with another. This problem has been solved before and in our case we can just use an Azure App Service to solve the problem.

![azure mobile architecture]({{ site.url }}/assets/posts/2017-09/azure-mobile-app-services-architecture.png)

Important items to note

* The App Code has <strong>1</strong> API it needs to program against
* As an app developer I am writing code in the `App Code` and maybe the `API` code, everything else is handled by the Azure Mobile App Services APIs
* Azure creates a local database for storing data while in offline mode
* When our app comes back online Azure provides simple APIs to handle the sync process

By adding a new layer into our system we have now simplified the app code to just using the Azure APIs which will handle all of our problems communicating with our local database and communicating with our service layer. 

## We Can't Use Azure ##
The most common thing I hear about using any of the tools Microsoft Azure puts out is "we can't use Azure because of X". Remember the phrase of this blog post is "Don't freak out" and tell your colleagues that you don't need Azure to use Mobile App Services. Having Microsoft Azure just makes it that much easier to get up and running.

<strong>Supported Servers</strong>

* Microsoft Hosted Azure
* Locally Hosted Azure
* IIS (yes, you can host this in house)

To host this in house you will deploy your OData service layer to your server just like you would do for anything else. For the purpose of this post we are going to talk about Azure since it makes our lives super easy.

## Configure Azure and Get Template ##
Configuring Azure and getting up and running is super easy if you follow the steps below. My favorite part about going through and configuring Azure for Mobile App Services is they provide code templates for almost every mobile platform and technology you will be working in from iOS Native (objective-c/swift) to Xamarin.Forms.

1. Log into your Azure Portal
2. Goto App Services
3. Click Add
4. Search for `Mobile App`
5. Create your new App Service
6. Once the App Service is created, navigate to the main page of the app
7. Under Deployment section select Quickstart
8. Select the platform you would like (we are going over Xamarin.Forms)
9. Connect your database
10. Create your backend service template
11. Create your client code template

### Azure Template ###
When I went through this process the template I downloaded did not work out of the box and instead of trying to get my environment to work exactly how Microsoft had it for the templates, I decided to update the template code to support .NET Standard and work for me. Feel free to download the code from my GitHub repo that I created as my starting point for Azure Mobile App Services

[https://github.com/surgeforward/AzureMobileAppServiceDemo](https://github.com/surgeforward/AzureMobileAppServiceDemo)

## Explore the Template ##
You should now have the client code and service code up and running locally and can start the template app. It is a simple Task Management App where you can add and remove tasks or as the app calls them `TodoItem`

![Azure mobile app template]({{ site.url }}/assets/posts/2017-09/azure-mobile-app-template.png)

<strong>Service</strong>

* Built on top of OData

<strong>Client Code</strong>

* Can use offline sync to retrieve data or regular RESTful calls
* `MobileServiceClient` easily allows you to specify your Service endpoint and start querying the backend

## Code Walkthrough and Breakdown ##
Let's walkthrough what I consider to be the important parts of the app that narrow down the App Service APIs and how to use them. Some of the APIs we will go into detail of what it is doing and others are just initialization code that we just need to call on startup or construction

### iOS Initialization ###
The only thing we added for iOS initialization is `CurrentPlatform.Init()`

{% highlight c# linenos %}
[Register("AppDelegate")]
public class AppDelegate : FormsApplicationDelegate
{
    public override bool FinishedLaunching(UIApplication app, NSDictionary options)
    {
        // Initialize Azure Mobile Apps
        CurrentPlatform.Init();

        // Initialize Xamarin Forms
        Forms.Init();

        LoadApplication(new App());

        return base.FinishedLaunching(app, options);
    }
}
{% endhighlight %}

### Android Initialization ###
The only thing we added for iOS initialization is `CurrentPlatform.Init()`

{% highlight c# linenos %}
[Activity(
        Label = "AzureMobileAppServiceDemo.Droid", 
        MainLauncher = true, 
        Icon = "@drawable/icon",
        ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation,
        Theme = "@android:style/Theme.Holo.Light")]
public class MainActivity : FormsApplicationActivity
{
    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);

        // Initialize Azure Mobile Apps
        CurrentPlatform.Init();

        // Initialize Xamarin Forms
        Forms.Init(this, bundle);

        // Load the main application
        LoadApplication(new App());
    }
}
{% endhighlight %}

### Client Code Shared Initialization ###
ToDoManager creates a new connection to the service layer to query the OData service. Other than this there is no Shared Code Initialization logic:

* `MobileServiceClient`
  - Creates the OData connection
* `client.GetTable<TodoItem>()`
  - Creates the object for manipulating the Entity where `TodoItem` is just a simple class of properties
* `todoTable` is our API table which is used to query the API

{% highlight C# linenos %}
public partial class TodoItemManager
{
    MobileServiceClient client;
    IMobileServiceTable<TodoItem> todoTable;

    private TodoItemManager()
    {
        client = new MobileServiceClient(Constants.ApplicationURL);
        todoTable = client.GetTable<TodoItem>();
    }
}
{% endhighlight %}

### Get Items ###
<strong>Simple GET</strong>
Querying the data tables is very straight forward and should look familiar

{% highlight C# linenos %}
public async Task<ObservableCollection<TodoItem>> GetTodoItemsAsync()
{
    var items = await todoTable
        .Where(todoItem => !todoItem.Done)
        .ToEnumerableAsync();
   return new ObservableCollection<TodoItem>(items);
}
{% endhighlight %}

<strong>Offline Sync GET</strong>
{% highlight C# linenos %}
public async Task<ObservableCollection<TodoItem>> GetTodoItemsAsync(bool syncItems = false)
{
    try
    {
        if (syncItems)
        {
            await this.SyncAsync();
        }

        IEnumerable<TodoItem> items = await todoTable
            .Where(todoItem => !todoItem.Done)
            .ToEnumerableAsync();

        return new ObservableCollection<TodoItem>(items);
    }
    catch (MobileServiceInvalidOperationException msioe)
    {
        Debug.WriteLine(@"Invalid sync operation: {0}", msioe.Message);
    }
    catch (Exception e)
    {
        Debug.WriteLine(@"Sync error: {0}", e.Message);
    }
    return null;
}
{% endhighlight %}

### Save Items ###
{% highlight C# linenos %}
public async Task SaveTaskAsync(TodoItem item)
{
    if (item.Id == null)
    {
        await todoTable.InsertAsync(item);
    }
    else
    {
        await todoTable.UpdateAsync(item);
    }
}
{% endhighlight %}

### Offline Sync Initialization ###
Setting up our project <strong>without</strong> offline sync is nice and all but we aren't really using the power that signed up for. To initialize your table objects for offline sync you need to run a few different commands including creating your local database.

* Create your serviceClient Connection
* Create your local database
* Add the Tables you want to store
* Initialize your syncContext
* Get your table

{% highlight C# linenos %}
MobileServiceClient client;
IMobileServiceSyncTable<TodoItem> todoTable;
const string offlineDbPath = @"localstore.db";

private TodoItemManager()
{
    client = new MobileServiceClient(Constants.ApplicationURL);
    var store = new MobileServiceSQLiteStore(offlineDbPath);
    store.DefineTable<TodoItem>();

    //Initializes the SyncContext using the default IMobileServiceSyncHandler.
    client.SyncContext.InitializeAsync(store);

    this.todoTable = client.GetSyncTable<TodoItem>();
}
{% endhighlight %}

### Offline Sync - How Does it Work ###
When it is time to actually sync we have the power to control it and handle errors. In the `TodoManager` there is the method `SyncAsync()` which handles the syncing of all the data that was added or removed while the app was offline

{% highlight C# linenos %}
public async Task SyncAsync()
{
    ReadOnlyCollection<MobileServiceTableOperationError> syncErrors = null;

    try
    {
        await this.client.SyncContext.PushAsync();

        await this.todoTable.PullAsync(
            //The first parameter is a query name that is used internally by the client SDK to implement incremental sync.
            //Use a different query name for each unique query in your program
            "allTodoItems",
            this.todoTable.CreateQuery());
    }
    catch (MobileServicePushFailedException exc)
    {
        if (exc.PushResult != null)
        {
            syncErrors = exc.PushResult.Errors;
        }
    }

    // Simple error/conflict handling. A real application would handle the various errors like network conditions,
    // server conflicts and others via the IMobileServiceSyncHandler.
    if (syncErrors != null)
    {
        foreach (var error in syncErrors)
        {
            if (error.OperationKind == MobileServiceTableOperationKind.Update && error.Result != null)
            {
                //Update failed, reverting to server's copy.
                await error.CancelAndUpdateItemAsync(error.Result);
            }
            else
            {
                // Discard local change.
                await error.CancelAndDiscardItemAsync();
            }

            Debug.WriteLine(@"Error executing sync operation. Item: {0} ({1}). Operation discarded.", error.TableName, error.Item["id"]);
        }
    }
}
{% endhighlight %}

### Service Layer - Owin Configuration ###
The Service Layer uses Owin which should be familiar, if it is not it is super easy to use

{% highlight C# linenos %}
public static void ConfigureMobileApp(IAppBuilder app)
{
    HttpConfiguration config = new HttpConfiguration();

    //For more information on Web API tracing, see http://go.microsoft.com/fwlink/?LinkId=620686 
    config.EnableSystemDiagnosticsTracing();

    // this is the configuration code you need to setup the Service layer
    new MobileAppConfiguration()
        .UseDefaultConfiguration()
        .ApplyTo(config);

    // Use Entity Framework Code First to create database tables based on your DbContext
    // for the purposes of our discussion this is just initializing Entity Framework, doesn't
    // really have a bearing on the azure mobile app service except it's name
    Database.SetInitializer(new AzureMobileAppServiceDemoInitializer());

    // To prevent Entity Framework from modifying your database schema, use a null database initializer
    // Database.SetInitializer<AppDbContext>(null);

    MobileAppSettingsDictionary settings = config.GetMobileAppSettingsProvider().GetMobileAppSettings();
{% endhighlight %}

### Service Layer - Entities ###
All the entities we create with Entity Framework should inherit from `EntityData` which implement `ITableData` which are important to remember. This is how the Azure tooling recgonizes our Entities

TodoItem.cs
{% highlight C# linenos %}
public class TodoItem : EntityData
{
    public string Text { get; set; }

    public bool Complete { get; set; }
}
{% endhighlight %}

EntityData.cs
{% highlight C# linenos %}
public abstract class EntityData : ITableData
{
    protected EntityData();

    public string Id { get; set; }
    public byte[] Version { get; set; }
    public DateTimeOffset? CreatedAt { get; set; }
    public DateTimeOffset? UpdatedAt { get; set;}
    public bool Deleted { get; set; }
}
{% endhighlight %}

### Service Layer - Controller Initialization ###

* Mobile Apps use the `TableController<T>` which provides the necessary hooks for offline sync and other features of Mobile Apps.
* Most of the code here is pretty standard for what you would see in an OData controller

{% highlight C# linenos %}
public class TodoItemController : TableController<TodoItem>
{
    protected override void Initialize(HttpControllerContext controllerContext)
    {
        base.Initialize(controllerContext);
        AppDbContext context = new AppDbContext();
        DomainManager = new EntityDomainManager<TodoItem>(context, Request);
    }

    // ...
}

{% endhighlight %}

### Service Layer - Controller OData Calls ###
Leveraging the `TableController<TodoItem>` API we can now handle our basic CRUD operations to retrieve our data and send it back up the pipe.

{% highlight C# linenos %}
public class TodoItemController : TableController<TodoItem>
{
    // ...

    // GET tables/TodoItem
    public IQueryable<TodoItem> GetAllTodoItems()
    {
        return Query();
    }

    // GET tables/TodoItem/48D68C86-6EA6-4C25-AA33-223FC9A27959
    public SingleResult<TodoItem> GetTodoItem(string id)
    {
        return Lookup(id);
    }

    // PATCH tables/TodoItem/48D68C86-6EA6-4C25-AA33-223FC9A27959
    public Task<TodoItem> PatchTodoItem(string id, Delta<TodoItem> patch)
    {
        return UpdateAsync(id, patch);
    }

    // POST tables/TodoItem
    public async Task<IHttpActionResult> PostTodoItem(TodoItem item)
    {
        TodoItem current = await InsertAsync(item);
        return CreatedAtRoute("Tables", new { id = current.Id }, current);
    }

    // DELETE tables/TodoItem/48D68C86-6EA6-4C25-AA33-223FC9A27959
    public Task DeleteTodoItem(string id)
    {
        return DeleteAsync(id);
    }
}
{% endhighlight %}

## Follow Up ##

We have covered the important parts of setting up Azure Mobile App Services and Offline Sync

There are APIs in Azure Mobile App Services that simplfy these common problems
* Authentication
* Push Notification

## Important Links ##

* [https://github.com/surgeforward/AzureMobileAppServiceDemo](https://github.com/surgeforward/AzureMobileAppServiceDemo)
* [https://azure.microsoft.com/](https://azure.microsoft.com/)
* [https://azure.microsoft.com/en-us/services/app-service/mobile/](https://azure.microsoft.com/en-us/services/app-service/mobile/)
* [https://azure.microsoft.com/en-us/documentation/learning-paths/appservice-mobileapps/](https://azure.microsoft.com/en-us/documentation/learning-paths/appservice-mobileapps/)




