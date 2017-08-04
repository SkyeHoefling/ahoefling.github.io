---
layout: post
title:  Entity Framework Mocking DbSet for Sync and Async Queries
modified: 2017-07-03 08:00:00
categories: dotnet
tags: [dotnet, Entity Framework, Testing, Moq, Async]
share: true
comments: true
---
When building a test framework it is sometimes useful to be able to write test code against `DbSet<T>` objects. Since `DbSet<T>` implements `IDbSet<T>` it is relatively easy to wire up a mock for your entity. Before we jump in let's go over an important concept in the Moq framework.

## Add Interface to Mock Object ##
To properly mock the `DbSet<T>` we will need to use the `As` function which adds an interface implementation to our mock. This funciton is critical when mocking out complex objects such as the `DbSet<T>`. The syntax is more or less the same as any mock but you are just attaching the interface implementation.

{% highlight c# linenos %}
// Suppose we have the interface IFoo and IBar where the method FooBar
// exists on IBar -> IBar.FooBar(). We can implement the IBar interface
// on the IFoo mock with the following code.
var mock = new Mock<IFoo>();
mock.As<IBar>().Setup(x => x.FooBar()).Returns("FooBar");
{% endhighlight %}

## Mock IQueryable of the DbSet (Sync) ##
After understanding how the simple mock works lets look at how we can handle the synchronous mocking of the `DbSet<T>` object. There are 4 parts to mocking the `IQueryable<T>`

1. Provider
2. Expression
3. ElementType
4. GetEnumerator()

This code will get you off and running with mocking out a synchronous `IDbSet<Foo>`
{% highlight c# linenos %}
var data = new [] 
{
    new Foo(),
    new Foo(),
    new Foo()
}.AsQueryable();

var mock = new Mock<IDbSet<Foo>>();
mock.As<IQueryable<Foo>>()
    .Setup(x => x.Provider)
    .Returns(data.Provider);

mock.As<IQueryable<Foo>>()
    .Setup(x => x.Expression)
    .Returns(data.Expression);

mock.As<IQueryable<Foo>>()
    .Setup(x => x.ElementType)
    .Returns(data.ElementType);
    
mock.As<IQueryable<Foo>>()
    .Setup(x => x.GetEnumerator())
    .Returns(data.GetEnumerator());
{% endhighlight %}

## Mock IQueryable of DbSet (Async) ##
Mocking the synchronous calls to the DbSet may be enough for your needs and if it is you are done. A lot of code is using async queries. This requires a little bit more work. Fortunately Microsoft has provided some great documentation on building Test Providers, Enumerators and Enumerables. [https://msdn.microsoft.com/en-us/library/dn314429(v=vs.113).aspx](https://msdn.microsoft.com/en-us/library/dn314429(v=vs.113).aspx)

1. Create your Test Providers, Enumerators and Enumerables
2. Mock everything you did for synchronous mocking
3. Add (`As`) interface implementation to your mock of `IDbAsyncEnumerable<T>`
4. Swap out your provider for the `TestDbAsyncQueryProvider<T>`

{% highlight c# linenos %}
var data = new [] 
{
    new Foo(),
    new Foo(),
    new Foo()
}.AsQueryable();

var mock = new Mock<IDbSet<Foo>>();

// This line is new
mock.As<IDbAsyncEnumerable<Foo>>()
    .Setup(x => x.GetAsyncEnumerator())
    .Returns(new TestDbAsyncEnumerator<Foo>(data.GetEnumerator()));

// this line is updated
mock.As<IQueryable<Foo>>()
    .Setup(x => x.Provider)
    .Returns(new Test DbAsyncQueryProvider<Foo>(data.Provider));

mock.As<IQueryable<Foo>>()
    .Setup(x => x.Expression)
    .Returns(data.Expression);

mock.As<IQueryable<Foo>>()
    .Setup(x => x.ElementType)
    .Returns(data.ElementType);

mock.As<IQueryable<Foo>>()
    .Setup(x => x.GetEnumerator())
    .Returns(data.GetEnumerator());
{% endhighlight %}

## Microsoft Test Classes ##
{% highlight c# linenos %}
internal class TestDbAsyncQueryProvider<TEntity> : IDbAsyncQueryProvider
{
    private readonly IQueryProvider _inner;

    internal TestDbAsyncQueryProvider(IQueryProvider inner)
    {
        _inner = inner;
    }

    public IQueryable CreateQuery(Expression expression)
    {
        return new TestDbAsyncEnumerable<TEntity>(expression);
    }

    public IQueryable<TElement> CreateQuery<TElement>(Expression expression)
    {
        return new TestDbAsyncEnumerable<TElement>(expression);
    }

    public object Execute(Expression expression)
    {
        return _inner.Execute(expression);
    }

    public TResult Execute<TResult>(Expression expression)
    {
        return _inner.Execute<TResult>(expression);
    }

    public Task<object> ExecuteAsync(Expression expression, CancellationToken cancellationToken)
    {
        return Task.FromResult(Execute(expression));
    }

    public Task<TResult> ExecuteAsync<TResult>(Expression expression, CancellationToken cancellationToken)
    {
        return Task.FromResult(Execute<TResult>(expression));
    }
}

internal class TestDbAsyncEnumerable<T> : EnumerableQuery<T>, IDbAsyncEnumerable<T>, IQueryable<T>
{
    public TestDbAsyncEnumerable(IEnumerable<T> enumerable)
        : base(enumerable)
    { }

    public TestDbAsyncEnumerable(Expression expression)
        : base(expression)
    { }

    public IDbAsyncEnumerator<T> GetAsyncEnumerator()
    {
        return new TestDbAsyncEnumerator<T>(this.AsEnumerable().GetEnumerator());
    }

    IDbAsyncEnumerator IDbAsyncEnumerable.GetAsyncEnumerator()
    {
        return GetAsyncEnumerator();
    }

    IQueryProvider IQueryable.Provider
    {
        get { return new TestDbAsyncQueryProvider<T>(this); }
    }
}

internal class TestDbAsyncEnumerator<T> : IDbAsyncEnumerator<T>
{
    private readonly IEnumerator<T> _inner;

    public TestDbAsyncEnumerator(IEnumerator<T> inner)
    {
        _inner = inner;
    }

    public void Dispose()
    {
        _inner.Dispose();
    }

    public Task<bool> MoveNextAsync(CancellationToken cancellationToken)
    {
        return Task.FromResult(_inner.MoveNext());
    }

    public T Current
    {
        get { return _inner.Current; }
    }

    object IDbAsyncEnumerator.Current
    {
        get { return Current; }
    }
}
{% endhighlight %}

## Putting It All Together ##
In my testing framework this is best put into an extension method so I can easily create this mock on the fly with as little code as possible. Here are some code snippets of how I would setup the extension method and how I would use it in a test.

Extension Method:
{% highlight c# linenos %}
public static class QueryableExtensions
{
    public static IDbSet<T> BuildMockDbSet<T>(this IQueryable<T> source)
        where T : class
    {
        var mock = new Mock<IDbSet<T>>();
        mock.As<IDbAsyncEnumerable<T>>()
            .Setup(x => x.GetAsyncEnumerator())
            .Returns(new TestDbAsyncEnumerator<T>(source.GetEnumerator()));

        mock.As<IQueryable<T>>()
            .Setup(x => x.Provider)
            .Returns(new TestDbAsyncQueryProvider<T>(source.Provider));

        mock.As<IQueryable<T>>()
            .Setup(x => x.Expression)
            .Returns(source.Expression);

        mock.As<IQueryable<T>>()
            .Setup(x => x.ElementType)
            .Returns(source.ElementType);

        mock.As<IQueryable<T>>()
            .Setup(x => x.GetEnumerator())
            .Returns(source.GetEnumerator());

        return mock.Object;
    }
}
{% endhighlight %}

Test Code:
{% highlight c# linenos %}
[TestFixture]
public class FooBarTests
{
    [Test]
    public void DbSetTest()
    {
        var data = new []
        {
            new Foo(),
            new Foo(),
            new Foo()
        }
        .AsQueryable()
        .BuildMockDbSet();

        // FooBarService takes in the `IDbSet<Foo>` 
        // All we have to do is pass the mocked object in
        // and we are able to write our test code with very
        // little build up
        var service = new FooBarService(data.Object);

        // test code would go here
    }
}

public class FooBarService
{
    public FooBarService(IDbSet<Foo> data)
    {
        // there would be real logic here that does
        // something with the constructor parameters
    }
}
{% endhighlight %}