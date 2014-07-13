---
layout: post
title: Blendable MVVM &#58; Dependency Injection and Unit Testing
tags: csharp silverlight
---

There are a couple of great reasons for using a UI separation pattern such as Model, View, ViewModel (or the more common MVP and MVC), the first being that with the application logic separated from the view logic the application becomes easier to maintain. Secondly it's usually easier for a designer and developer to work on the separate areas and **integrate** them at a later date (this actually helps enforce the separation). And thirdly by having the application logic away from the UI it becomes much easier to test.

While you can do UI testing with the Silverlight testing framework it becomes a hell of a lot easier when you have a nice separation. We're doing to be using a dependency injection framework (in this caseNinject) to build the view models and inject the services. This'll allow us to do  **interaction based unit testing** on the view model using Rhino Mocks.

Overall our goal will be to refactor the existing ViewModel to break it's dependency on CocktailsService and increase it's testability. We'll create a Service Locator that will be an Application resource and serve as almost a "**ViewModel factory**" and then instead of directly creating our ViewModel in xaml we'll bind our Data Context to the Service Locator.

Our first step will be inverting the dependency the ViewModel has on the Cocktails Service, we'll extract an interface for the service and use constructor injection to pass the service to the ViewModel where it'll be stored, otherwise the view model remains the same.

``` csharp
public interface ICocktailService
{
    IEnumerable<Cocktail> GetCocktails();
    IEnumerable<Cocktail> GetCocktailsSimilarTo(Cocktail cocktail);
}

public CocktailsViewModel(ICocktailService cocktailService)
{
    this.cocktailService = cocktailService;
 
    AvailableCocktails = new ObservableCollection<Cocktail>();
    SimilarCocktails = new ObservableCollection<Cocktail>();
 
    GetSimilarCocktailsCommand = new DelegateCommand<Cocktail>(GetSimilarCocktails);
 
    AvailableCocktails.AddRange(cocktailService.GetCocktails());
}
```
We can now test that the ViewModel exposes the available Cocktails, passes the command parameter to the service and exposes the similar cocktails.

``` csharp
[TestClass]
public class CocktailsViewModelFixture
{
    private readonly List<Cocktail> sampleCocktails = new List<Cocktail>
        {
            new Cocktail
            {
                Id = 1,
                Ingredients = new string[] {},
                Name = "Test"
            }
        };
 
    [TestMethod]
    public void ConstuctionRetrievesAvailableCockstails()
    {
        var cocktailService = MockRepository.GenerateMock<ICocktailService>();
 
        cocktailService.Expect(c => c.GetCocktails()).Return(Enumerable.Empty<Cocktail>());
 
        new CocktailsViewModel(cocktailService);
 
        cocktailService.VerifyAllExpectations();
    }
 
    [TestMethod]
    public void ConstuctionExposesAvailableCockstails()
    {
        var cocktailService = MockRepository.GenerateStub<ICocktailService>();
 
        cocktailService.Stub(c => c.GetCocktails()).Return(sampleCocktails);
 
        var viewModel = new CocktailsViewModel(cocktailService);
 
        Assert.AreEqual(sampleCocktails.Count, viewModel.AvailableCocktails.Count);
    }
 
    [TestMethod]
    public void GetSimilarCocktailsForwardsCommandParameter()
    {
        var selectedCocktail = new Cocktail();
 
        var cocktailService = MockRepository.GenerateMock<ICocktailService>();
 
        cocktailService.Stub(c => c.GetCocktails()).Return(Enumerable.Empty<Cocktail>());
        cocktailService.Expect(c => c.GetCocktailsSimilarTo(selectedCocktail)).Return(Enumerable.Empty<Cocktail>());
 
        var viewModel = new CocktailsViewModel(cocktailService);
 
        viewModel.GetSimilarCocktailsCommand.Execute(selectedCocktail);
 
        cocktailService.VerifyAllExpectations();
    }
 
    [TestMethod]
    public void GetSimilarCocktailsExposesSimilarCocktails()
    {
        var selectedCocktail = new Cocktail();
 
        var cocktailService = MockRepository.GenerateMock<ICocktailService>();
 
        cocktailService.Stub(c => c.GetCocktails()).Return(Enumerable.Empty<Cocktail>());
        cocktailService.Stub(c => c.GetCocktailsSimilarTo(selectedCocktail)).Return(sampleCocktails);
 
        var viewModel = new CocktailsViewModel(cocktailService);
 
        viewModel.GetSimilarCocktailsCommand.Execute(selectedCocktail);
 
        Assert.AreEqual(sampleCocktails.Count, viewModel.SimilarCocktails.Count);
    }
}
```

Astute readers will notice that because our ViewModel doesn't have a parameterless constructor we can't create it directly in xaml. This is correct, instead we'll be delegating the construction of the ViewModel to a Ninject kernel. We'll create a ServiceLocator that will be an Application Resource (in App.xaml), it will create the Ninject kernel and register the appropriate dependencies.

``` csharp
public class ServiceLocator
{
    private readonly IKernel kernal;
 
    public ServiceLocator()
    {
        kernal = new StandardKernel(new CocktailModule());
    }
 
    public CocktailsViewModel CocktailsViewModel
    {
        get
        {
            return kernal.Get<CocktailsViewModel>();
        }
    }
}

public class CocktailModule : StandardModule
{
    public override void Load()
    {
        Bind<CocktailsViewModel>().ToSelf();
        Bind<ICocktailService>().To<CocktailService>();
    }
}
```

``` xml
<Application.Resources>
    <cx:ServiceLocator x:Key="ServiceLocator"/>
</Application.Resources>
```

For simplicities sake I have a simple module that will register the dependencies we need.

Unfortunately there's no nice GUI to hook the ViewModel up anymore we have to write our own Binding expression, but it's not too bad. This binds the DataContext of the UserControl to the property "CocktailsViewModel" which is in turn created by the Ninject kernal, injecting the appropriate dependencies.

```
DataContext="{Binding Path=CocktailsViewModel,Source={StaticResource ServiceLocator}}"
```
