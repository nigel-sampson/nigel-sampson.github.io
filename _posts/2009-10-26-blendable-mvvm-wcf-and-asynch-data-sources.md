---
layout: post
title: Blendable MVVM&#58; WCF and Asynch Data Sources
tags: wcf csharp silverlight
---

In the previous posts we've been retrieving our data from a synchronous data source, for the most part in Silverlight we'll be retrieving data sources from aseparate server, in this case through a WCF Service. Silverlight has a restriction that all network calls must be asynchronous in order to avoid locking up the browser UI (a good thing). It does however make the code a little more complicated.

The functionality of the CocktailsService will stay the same, we're just going to migrate it over to the server and host it as a WCF service. The code for the service is remains exactly the same, just shifted now to the server and hosted as a WCF service. If you'd like to learn more about setting up a WCF service you can read more on [MSDN](http://msdn.microsoft.com/en-us/magazine/cc794260.aspx).

``` csharp
[ServiceContract]
public interface ICocktailService
{
    [OperationContract]
    IEnumerable<Cocktail> GetCocktails();
 
    [OperationContract]
    IEnumerable<Cocktail> GetCocktailsSimilarTo(Cocktail cocktail);
}
```

Then we generate a client proxy using Visual Studios "Add Service Reference". The proxy generated includes any types from the data contracts (the Cocktail class), the proxy client (CocktailsServiceClient) and a service interface (ICocktailsService). One thing to notice if you've done any WCF work before is that the interface doesn't contain the events exposed by the actual client class.

We'll now need to alter our Cocktails view model to deal with the new interface. Basically it's a relatively simple change from synchronous methods to using async ones. It should look like this...

``` csharp
public class CocktailsViewModel : ViewModelBase<CocktailsViewModel>
{
    private readonly ICocktailService cocktailService;
 
    public CocktailsViewModel(ICocktailService cocktailService)
    {
        this.cocktailService = cocktailService;
 
        AvailableCocktails = new ObservableCollection<Cocktail>();
        SimilarCocktails = new ObservableCollection<Cocktail>();
 
        GetSimilarCocktailsCommand = new DelegateCommand<Cocktail>(GetSimilarCocktails);
 
        cocktailService.BeginGetCocktails(OnGetCocktails, null);
    }
 
    private void OnGetCocktails(IAsyncResult ar)
    {
        var cocktails = cocktailService.EndGetCocktails(ar);
 
        AvailableCocktails.AddRange(cocktails);
    }
 
    public ObservableCollection<Cocktail> AvailableCocktails
    {
        get;
        set;
    }
 
    public ObservableCollection<Cocktail> SimilarCocktails
    {
        get;
        set;
    }
 
    public ICommand GetSimilarCocktailsCommand
    {
        get;
        private set;
    }
 
    protected void GetSimilarCocktails(Cocktail cocktail)
    {
        cocktailService.BeginGetCocktailsSimilarTo(cocktail, OnGetCocktailsSimilarTo, null);
    }
 
    private void OnGetCocktailsSimilarTo(IAsyncResult ar)
    {
        var cocktails = cocktailService.EndGetCocktailsSimilarTo(ar);
 
        SimilarCocktails.Replace(cocktails);
    }
}
```

There is a slight problem with the code above. Silverlight expects property changed notifications (including collection changed from ObservableCollection) to be on the UI thread. The async result methods will be on a separate thread (The generated service client will dispatch the OnXCompleted events back to the UI thread but not the asynch methods). There a couple of ways we can solve this.

1. Manually dispatch any updates to properties back to the UI thread.
2. Create our own ObservableCollection that ensures all notifications are on the correct thread. An example of this can be found at "[Adding to an ObservableCollection from a background thread](http://www.julmar.com/blog/mark/2009/04/01/AddingToAnObservableCollectionFromABackgroundThread.aspx)"

For this example we'll just be going with the first. We'll update our ViewModelBase to take a reference to the Application dispatcher and have a simple helper method.

``` csharp
protected ViewModelBase()
{
    Dispatcher = Deployment.Current.Dispatcher;
}
 
protected Dispatcher Dispatcher
{
    get; private set;
}
 
protected void InvokeOnUIThread(Action action)
{
    Dispatcher.BeginInvoke(action);
}
```

We then alter the offending methods to use the helper.

``` csharp
private void OnGetCocktails(IAsyncResult ar)
{
    var cocktails = cocktailService.EndGetCocktails(ar);
 
    InvokeOnUIThread(() => AvailableCocktails.AddRange(cocktails));
}

private void OnGetCocktailsSimilarTo(IAsyncResult ar)
{
    var cocktails = cocktailService.EndGetCocktailsSimilarTo(ar);
 
    InvokeOnUIThread(() => SimilarCocktails.Replace(cocktails));
}
```

Our updates to the collections will be dispatched off to the UI thread and the exception will be avoided.
