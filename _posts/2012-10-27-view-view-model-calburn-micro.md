---
layout: post
title: View or View Model first in Caliburn Micro WinRT
tags: csharp caliburn.micro windows-apps windows-phone
---

[Caliburn Micro][caliburn] on its various platforms has usually supported either a View Model first or a View first [approach][approaches], but not usually both at the same time. Typically Silverlight and WPF applications follow a View Model first approach, usually with a Shell View Model and using view model [composition][composition]. Meanwhile [Windows Phone][phone] applications due to having the navigation concept baked very close to the hardware (with the back button) typically follow a View first approach and expose a navigation service to move between pages.

Windows 8 apps sit somewhere in the middle, since there is no hardware back button we don’t have the same drive towards View first, however most apps follow a design where a root Frame control navigating between pages makes sense. However there is nothing stopping a developer using a View Model first approach and for certain apps and scenarios this makes sense. In fact I feel that fully featured apps will use both approaches.

The latest commit to Caliburn Micro on [Codeplex][caliburn] (and hopefully published to Nuget soon) adds support for both approaches out of the box, a first for the library. This adds some breaking changes so I thought I’d write a bit more about them and how it affects your apps.

The initial versions of Caliburn Micro for WinRT did in my opinion a little too much around launch, while GetDefaultView works well for a first app it quickly becomes cumbersome when you want to launch different views based on launch arguments or when the app is activated for Search etc. Because of this the new version steps back and lets you override the appropriate methods in the Application and direct Caliburn Micro what to do. This hands back the control to you and should mean that framework gets out of your way when you need it to.

Here’s some examples of how you’d run through each scenario.

###View First
This approach is what you’re used to if you’ve been using the WinRT version up until now, it’s fundamentally the same with a few minor changes in your App.xaml.cs. The first thing to note is that WinRTContainer.RegisterDefaultServices doesn’t register an instance of INavigationService as it wouldn’t make sense in View Model first scenarios. Instead we override a method named PrepareViewFirst that has a parameter of the root frame for the application (this is also accessible through the RootFrame property). We can then pass this to WinRTContainer.RegisterNavigationService, this creates the required FrameAdapter and registers it to the container as a navigation service. If you’re using a different container this is where you’d do the same with your own container.

``` csharp
protected override void PrepareViewFirst(Frame rootFrame)
{
    container.RegisterNavigationService(rootFrame);
}
```

Now instead of defining the default view we’ll override OnLaunched, this is the method called by Windows 8 on launch. Here we’ll call DisplayRootView with the type of the view we want our root frame to navigate to, in this case MenuView. This approach enables us to use things like the launch arguments and choose a different view to navigate to. Caliburn will ensure then ensure the Bootstrapper is initialized, set the root frame as the content of the window and navigate to the specified view.

``` csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    DisplayRootView<MenuView>();
}
```

This should be the same behavior as before these changes.

###View Model First
This approach takes advantage of the view model [composition][composition] built into Caliburn Micro. It’s a little simpler to set up than view first. We don’t need to override PrepareViewFirst and in our OnLaunched we call DisplayRootViewFor with the view model as the type. Caliburn determines the view as per its conventions and sets that as the content of the window as well as binding the two together.

``` csharp
protected override void OnLaunched(LaunchActivatedEventArgs args)
{
    DisplayRootViewFor<ShellViewModel>();
}
```

Remember that in a view model first approach the INavigationService will not be available to use.

###Things to watch out for
Due to the event model of a WinRT application there isn’t a suitable place to initialize the CaliburnApplication before OnLaunched, therefore both DisplayRootView and DisplayRootViewFor will Initialize the application (and ultimately call Configure). If you’re dependent upon the application already being configured then you can call Initialize yourself.

###Combining both approaches
Any significantly sized WinRT application will most likely use a combination of the two approaches. While you may use View first to quickly enable the navigation metaphor with a back button you may compose individual views with child view models. Also some launch scenarios such as Share Target give you a small window to work in where View Model first will be simpler. The sample WinRT application shows this approach.

Overall these changes should give you increased functionality and better control of your application using Caliburn Micro.

[caliburn]: http://caliburnmicro.codeplex.com
[approaches]: http://caliburnmicro.codeplex.com/wikipage?title=All%20About%20Conventions&referringTitle=Documentation
[composition]: http://caliburnmicro.codeplex.com/wikipage?title=Screens%2c%20Conductors%20and%20Composition&referringTitle=Documentation
[phone]: http://caliburnmicro.codeplex.com/wikipage?title=Working%20with%20Windows%20Phone%207%20v1.1&referringTitle=Documentation
