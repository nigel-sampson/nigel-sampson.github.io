---
layout: post
title: Views for types that aren't View Models
tags: csharp caliburn.micro windows-apps windows-phone silverlight wpf
---

In most MVVM style frameworks we're used of seeing pairs of views and view models, RepositoryView and RepositoryViewModel for instance. We may sometimes have view models without views (such as GroupViewModel) but how do we deal with instances when we want views for types that aren't view models? 

I'm currently redesigning and rebuilding bits of [Hub Bug][hubbug] and making use of the [Octokit.NET][octokit] library from GitHub. This awesome nuget package provides 90% of what I'd call my model.  Often view models wrap these model types when bound to the view. Yet some of the smaller types such as *Label* and *User* may be bound directly to the view inside list boxes or form small parts of a large view. Creating view models just so Caliburn.Micro's **ViewLocator** can locate the correct view feels like a waste as the view models don't add any functionality and create unnecessary ceremony. 

By default Caliburn.Micro won't know what to do with these types, they don't end in "ViewModel" so it won't find any views. Even if it did it would try to locate them in the Octokit assembly which definitely isn't something we want. 

What we want to do is give the ViewLocator a hint that when it tries to locate views for certain types to look somewhere where it wouldn't normally look. In this case When it locates a view for **Octokit.User** I want to it to use **HubBug.App.Views.Octokit.UserView**. In application set up (either the Bootstrapper or the Application depending on platform) I have the following:

```csharp
ViewLocator.NameTransformer.AddRule(
    @"^Octokit\.(\w*)",
    @"HubBug.App.Views.Octokit.${1}View");
```

This uses lovely old regular expressions to transform the view model type to the view type, that's all it takes.

What's very cool about this is that I now have a consistent way to display Users, whether I'm binding to a ComboBox or it's part of a Grid. Caliburn makes sure I'm using the correct view in every instance. 

For instance any where I want to display a user I can drop in the following xaml, tweak the binding appropriate to the view and I'm done. 

```xml
<ContentControl micro:View.Model="{Binding Repository.Owner}" FontSize="{StaticResource ControlContentThemeFontSize}"/>
```

ViewLocator has quite a few customisation options, take a look and see how you can streamline your app development.

[octokit]: http://octokit.github.io/
[hubbug]: http://compiledexperience.com/windows-apps/hub-bug
[cm]: https://github.com/BlueSpire/Caliburn.Micro
