---
layout: post
title: Convention Based Event Aggregation in WPF and Silverlight Applications 
tags: csharp silverlight wpf caliburn
---

One thing that can get pretty tedious very quickly in building composite UI's in Silverlight and WPF is wiring up an event aggregator. Typically if a service or view model wants to subscribe to an event in the application you would declare a dependency on IEventAggregator and call Subscribe on it passing the method to invoke when the event is published.

The Event Aggregator we'll be using for this example is based off something like the "[Suck Less Event Aggregator](http://blogs.msdn.com/ukadc/archive/2009/12/08/a-suck-less-event-aggregator-for-prism.aspx)" and the introduction of non generic methods required for this example. The interface for this example looks as follows.

``` csharp
public interface IEventAggregator
{
    void Subscribe<T>(Action<T> action);
    void Subscribe<T>(Action<T> action, bool keepSubscriberAlive);
 
    void Subscribe(Type eventType, Delegate action);
    void Subscribe(Type eventType, Delegate action, bool keepSubscriberAlive);
 
    void Unsubscribe<T>(Action<T> action);
    void Unsubscribe(Type eventType, Delegate action);
 
    void Publish<T>(T eventToPublish);
}
```

This seems pretty standard interface the problem is that over a whole applications worth of services and view models it's a lot of repetitious code to subscribe to all the events we're interested in. Lots of the view models will have dependencies on the event aggregator which could end mean a lot of repetitious unit tests.

What I'm going to try and show today is how to use some conventions to simplify your code. We'll adopt the convention that any public method on a View Model named **"OnX**" on a and has one parameter should be subscribed to an event (the type of the event will be based on the parameter type).

I'm currently using [Caliburn](http://caliburn.codeplex.com) to build my WPF applications but Prism is the same in this regard. Both don't have an explicit way to construct a new View Model, they simply want you to depend on the IServiceLocator or IUnityContainer and use it to Resolve your new ViewModel. Since we'll be wanting to insert some functionality into this process we'll create our own <strong>IViewModelFactory</strong> it'll be pretty simple and just shift the dependency on IServiceLocator into a specific place. The interface and implementation are.

``` csharp
public interface IViewModelFactory
{
    T Create<T>() where T : IPresenter;
}
 
public class DefaultViewModelFactory : IViewModelFactory
{
    private readonly IServiceLocator serviceLocator;
 
    public DefaultViewModelFactory(IServiceLocator serviceLocator)
    {
        if(serviceLocator == null)
            throw new ArgumentNullException("serviceLocator");
 
        this.serviceLocator = serviceLocator;
    }
 
    public T Create<T>() where T : IPresenter
    {
        var viewModel = serviceLocator.GetInstance<T>();
 
        return viewModel;
    }
}
```

We'll use the [Decorator Pattern](http://en.wikipedia.org/wiki/Decorator_pattern) to add extra functionality to our View Model factory, our **EventAggregatorViewModelFactory** takes a dependency on another **IViewModelFactory** and uses that to actually create the viewModel, we'll simply do some extra work on the result before passing back to the caller. The Decorator pattern is great in this regard in that I can plug even more functionality into the pipeline of view model creation without requiring each piece of functionality to know about any of the others.

``` csharp
public class EventAggregationViewModelFactory : IViewModelFactory
{
    private readonly IViewModelFactory viewModelFactory;
    private readonly IEventAggregator eventAggregator;
 
    public EventAggregationViewModelFactory(IViewModelFactory viewModelFactory, IEventAggregator eventAggregator)
    {
        this.viewModelFactory = viewModelFactory;
        this.eventAggregator = eventAggregator;
    }
 
    public T Create<T>() where T : IPresenter
    {
        var viewModel = viewModelFactory.Create<T>();
 
        AttachSubscriptions(viewModel);
 
        return viewModel;
    }
 
    protected void AttachSubscriptions<T>(T viewModel) where T : IPresenter
    {
        var methods = from m in viewModel.GetType().GetMethods(BindingFlags.Public | BindingFlags.Instance)
                      where IsEventSubscriptionMethod(m)
                      select m;
 
        foreach(var method in methods)
        {
            var eventType = method.GetParameters()[0].ParameterType;
            var delegateType = Expression.GetActionType(eventType);
            var @delegate = Delegate.CreateDelegate(delegateType, viewModel, method);
 
            eventAggregator.Subscribe(eventType, @delegate);
        }
    }
 
    protected virtual bool IsEventSubscriptionMethod(MethodInfo method)
    {
        return method.Name.StartsWith("On") && method.GetParameters().Length == 1;
    }
}
```

AttachSubscriptions is where the extra functionality is added, we first find all methods on the ViewModel that meet the convention (note the logic is in a method that can be overridden to change the convention). Once we have the methods we're interested in we create a delegate&nbsp; (Action&lt;T&gt;) based on the event type and the method, we then call subscribe on the event aggreator using the event type and our delegate. Pretty simple really, the only complicated part is that most event aggregators don't come with a non generic interface, this could be avoided if you're ok with using reflection to invoke one of the generic methods instead.

I'm wiring the entire thing together using Ninject and the following code.

``` csharp
kernel.Bind<IViewModelFactory>().To<EventAggregationViewModelFactory>().InSingletonScope();
kernel.Bind<IViewModelFactory>().To<DefaultViewModelFactory>().WhenInjectedInto<EventAggregationViewModelFactory>().InSingletonScope();
```

With this in place we can now simplify an example view model from the following

``` csharp
public class DependentCustomerViewModel : Presenter
{
    private readonly IEventAggregator eventAggregator;
 
    public DependentCustomerViewModel(IEventAggregator eventAggregator)
    {
        this.eventAggregator = eventAggregator;
    }
 
    protected override void OnInitialize()
    {
        base.OnInitialize();
 
        eventAggregator.Subscribe<CustomerCreatedEvent>(OnCustomerCreated);
    }
 
    protected void OnCustomerCreated(CustomerCreatedEvent customerCreatedEvent)
    {
        // Business logic for customer creation
    }
}
```

to

``` csharp
public class ConventionCustomerViewModel : Presenter
{
    public void OnCustomerCreated(CustomerCreatedEvent customerCreatedEvent)
    {
        // Business logic for customer creation
    }
}
```
