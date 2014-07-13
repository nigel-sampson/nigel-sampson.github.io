---
layout: post
title: Caliburn 1.1 on Windows Phone 7
tags: 
  - csharp
  - silverlight
  - windows-phone
  - caliburn
---

I've always enjoyed working with the [Caliburn](http://caliburn.codeplex.com/) framework for WPF and Silverlight, having the view and view model wired with so many conventions makes creating a rich view model really easy. So one of the first things I did after getting a hold of the Windows Phone 7 CTP was to try and get Caliburn working.

On a side note Rob Eisenberg has released [Caliburn Micro](http://caliburnmicro.codeplex.com/) which is a slimmed down version of Caliburn that also targets WP7. It's a great little library but has some problems in the WP7 version due to the incomplete binding infrastructure in WP7 (lack of binding on anything not subclassing FrameworkElement), this makes Action Parameters impossible. The original Caliburn doesn't have this problem due to a more complicated code base.

Getting Caliburn 1.1 to compile for WP7 wasn't too difficult, some features like DefaultWindowManager were dropped and since we don't have support for System.Reflection.Emit then some of the code in DelegateFactory didn't work. In the end I replaced with some anonymous methods using simple reflection. Not as elegant and probably not as performant.

As a bonus I've included the [Ninject](http://ninject.org/) library and Caliburn Ninject adapter.

So how do get this whole thing going? First we have to look at some of the ways the Windows Phone 7 framework expects things to work and how it impacts Caliburn. These also affect Caliburn Micro as well.

The major one is that WP7 expects the RootVisual to be a PhoneApplicationFrame, this provides a lot of the navigation support as well as Orientations. You can circumvent this but from comments from Microsoft you'll be fighting the framework the entire way (for example WP7 will close an application that hasn't navigated within the first ten seconds). If you end up supporting the navigation metaphor essentially you're now using a "view first" approach rather than the default Caliburn approach of "view model first".

``` csharp
public partial class ShellView
{
    private readonly IServiceLocator serviceLocator;
    private readonly IBinder binder;
 
    public ShellView(IServiceLocator serviceLocator, IBinder binder)
    {
        this.serviceLocator = serviceLocator;
        this.binder = binder;
 
        InitializeComponent();
 
        Loaded += OnLoaded;
        Navigated += OnNavigated;
    }
 
    private void OnLoaded(object sender, RoutedEventArgs e)
    {
        // I prefer this to start navigation than the _default Task in WMAppMainfest.xml
        Navigate(new Uri("/Views/MainPageView.xaml", UriKind.Relative));
    }
 
    private void OnNavigated(object sender, NavigationEventArgs e)
    {
        var viewModel = BuildViewModel(e);
 
        binder.Bind(viewModel, e.Content, null);
 
        var presenter = viewModel as IPresenter;
 
        if(presenter == null)
            return;
 
        presenter.Initialize();
        presenter.Activate();
    }
 
    private object BuildViewModel(NavigationEventArgs e)
    {
        var viewType = e.Content.GetType();
 
        var viewModelTypeName = viewType.FullName.Replace("View", "ViewModel");
        var viewModelType = Type.GetType(viewModelTypeName);
 
        return serviceLocator.GetInstance(viewModelType);
    }
}
```

Unlike Silverlight 4 we can't alter the content loader in PhoneApplicationFrame, so for navigation we'll need to handle when the frame changes it's current page and wire in a view model at this state. We can do this via a "view model locator" in the same faction as the default "view locator", but for the moment we'll manually wire the view and the view model. Caliburn Micro has slightly better support for the "view first" approach and I'll tackle that next. Below is a link containing the assemblies and a sample project.

[Download Caliburn 1.1 for Windows Phone 7](/content/downloads/caliburn.phone.zip)
