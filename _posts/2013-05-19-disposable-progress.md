---
layout: post
title: Disposable Progress Indicators
tags: csharp windows-phone windows-apps
---

Often in your view models you’ll need to indicate to the view that some work is happening. In fact I would mandate it, for low powered Windows RT devices. These add to the idea of “perceived performance” in that something on the screen reacts immediately to the users touch and they’re not mashing the screen wondering if it’s working.

I typically make a distinction between Loading and Working. For me Loading is creating, requesting the data that the view presents and would normally be indicated by a Progress Ring in the part of the view this data would be normally shown. Working tends to come after this when the user is acting on the data already loaded, such as Closing an Issue in Hub Bug which I tend to use a Progress Bar across the top of the app (much akin to the Progress Indicator in Windows Phone which tends to get used for both).

There’s a lot of different ways to approach this problem, some people use visual states others use simple Boolean properties on their view models. One thing I found myself doing a lot was starting the working or loading indicator but forgetting to stop it. Admittedly this was pre async / await where you had to bury the stop code in a lambda or callback.

I’ve found using (abusing?) the using statement with disposables to create Loading or Working blocks is a nice way to structure view models so it’s very easy to see how that UI is affected by the view model.

The first thing we need is a little helper class named DisposableAction, this class simply implements IDisposable and takes an Action. When the class is disposed it calls the Action.

```csharp
public class DisposableAction : IDisposable
{
    private readonly Action action;
 
    public DisposableAction(Action action)
    {
        this.action = action;
    }
 
    public void Dispose()
    {
        action();
    }
}
```

For this example I’ll just a Boolean property IsLoading that’s hypothetically bound to the visibility of a Progress Ring. We’ll create a method calling Loading that first sets IsLoading to true and then returns an DisposableAction that sets it to false when it’s disposed.

``` csharp
protected IDisposable Loading()
{
    IsLoading = true;
 
    return new DisposableAction(() => IsLoading = false);
}
```

Now that we have our Loading method we can use it in loading our view model (in this case [Caliburn.Micro][cm]'s OnInitiated). Because Loading returns an IDisposable we can use the using statement to wrap our code and quickly visualise how the Loading.

``` csharp
protected override async void OnInitialize()
{
    base.OnInitialize();
 
    using (Loading())
    {
        await BindCategoriesAsync();
        await BindProductsAsync();
    }
}
```

[cm]: https://github.com/BlueSpire/Caliburn.Micro
