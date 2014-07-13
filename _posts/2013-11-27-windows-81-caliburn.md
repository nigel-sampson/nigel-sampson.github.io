---
layout: post
title: Windows 8.1 and Caliburn.Micro
tags: csharp caliburn.micro windows-apps
---

In the last few months Windows 8.1 has been released and I’ve had a few questions about Caliburn.Micro and where it stands. Also Caliburn.Micro is releasing an alpha of a new version [announced by Rob][announce].

The current version of CM for Windows 8 will work with Windows 8.1 with one issue that I know of. In 8.0 we didn’t have access to the binding infrastructure, especially GetBindingExpression which is how CM determines if a property is already bound. We had to use a pretty nasty hack to get around this, with the changes in 8.1 this nasty hack doesn’t work. 

What this means is that if you have an element that could have a convention applied to it by CM but there was already a binding in place CM under 8.0 would skip applying the convention and respect your binding, under 8.1 this won’t happen and CM will happily overwrite the binding with the convention. Not a nasty bug as long as you’re aware of it.

Thankfully in 8.1 we can now do away with our nasty hack and new versions should work well. Speaking of new versions, the current git repository on Codeplex contains the code for the 8.1 version of CM which is still undergoing testing but it doesn’t hurt to take a look at. 

From the 8.1 perspective it includes the following:

 - Removal of the Callisto dependency in favor of the new Settings Flyout control.
 - Removal of the Windows.UI.Interactivity dependency in favor of the new Behaviours SDK from Microsoft.
 - Conventions for the new controls such as DatePicker, Flyout, CommandBar and many more.
 - Usage of the underlying improvements to the xaml framework for binding including UpdateSourceTrigger.

It’s also available as a pre-release package in [Nuget][nuget].

Please bear in mind that at the same time CM is making a more to a more portable approach, therefore to keep up with Semantic Versioning this next release will be 2.0 and will definitely contain breaking changes.

[announce]: http://devlicio.us/blogs/rob_eisenberg/archive/2013/11/25/caliburn-micro-2-0-0-alpha2-is-here.aspx
[nuget]: http://www.nuget.org/packages/Caliburn.Micro/
