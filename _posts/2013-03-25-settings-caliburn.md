---
layout: post
title: WinRT Settings with Caliburn Micro
tags: csharp caliburn.micro windows-apps
---

As well as support for the [Share charm][sharing], the recent [1.5 release of Caliburn Micro][release] adds to functionality around the Settings charm as well!

The built in Xaml framework for WinRT doesn’t really help developers for this charm which is a shame, there’s no “settings flyout control” and obviously no MVVM support at all. Thankfully some awesome libraries like [Callisto][callisto] have stepped up to provide the missing controls and Caliburn Micro now integrates them into the conventions you’re used to.

To enable support for the new Settings functionality we call the method **RegisterSettingsService** on **WinRTContainer**. This method returns an **ISettingsService** which we can use to register command, these commands represent options presented to the user when they invoke the Settings Charm (Windows + I as a shortcut).

``` csharp
var settings = container.RegisterSettingsService();
```

For Caliburn Micro each command will be represented by both a view model and a label. The label is the test displayed to the user by the Settings Charm while the view model represents the view / view model combination that will be displayed to the user if that command is selected. As with all view models in Caliburn Micro these need first need to be registered with the container itself.

``` csharp
container
    .PerRequest<SettingsViewModel>()
    .PerRequest<AboutViewModel>()
    .PerRequest<PrivacyPolicyViewModel>();
```

Once registered with the container you can register them with the settings service.

``` csharp
settings.RegisterCommand<SettingsViewModel>("Settings");
settings.RegisterCommand<AboutViewModel>("About");
settings.RegisterCommand<PrivacyPolicyViewModel>("Privacy Policy");
```

By default when the user selects the command Caliburn Micro will create a Callisto Settings Flyout control, locate the view for that specified view model placing it within the flyout and binding the view and view model together enabling the conventions and binding you already know and love!

As I mentioned above Caliburn Micro uses Callisto as the Settings Window Manager but this is abstracted behind the **ISettingsWindowManager** interface which you can implement yourself if you wish to use some other mechanism.

With this functionality you can easily create view / view model pairs to represent your settings pages and hand off the wiring to Caliburn Micro!

[release]: http://devlicio.us/blogs/rob_eisenberg/archive/2013/03/18/durandal-1-2-0-and-caliburn-micro-1-5-0-released.aspx
[sharing]: http://compiledexperience.com/blog/posts/share-caliburn
[callisto]: https://github.com/timheuer/callisto
