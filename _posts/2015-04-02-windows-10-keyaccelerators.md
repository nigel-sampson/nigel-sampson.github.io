---
layout: post
title: What's new in Windows 10 - Key Accelerators
tags: windows-apps windows-phone  
---

Microsoft have finally started to show off what's available in Windows 10 for us app developers. Over the next few days we can start to take a look at some of these changes and what it can mean. Today we'll look at what you can do with the `Windows.UI.Xaml.KeyAccelerator` class.

## Disclaimer

This was written using the initial release of the Windows 10 Developer Tools running on Windows 10 build 10041 and things will mostly change before final release. I'll try to come back and keep these posts up to date as time goes on.

## Key Accelerator
Keyboard shortcuts aren't thought much about by app developers, we typically envisage our app running on a touch enabled tablet or phone. Although Windows 8 apps would run with no problems on desktop devices they weren't thought of as a target. Windows 10 by supporting *Desktop Mode* and the *Continuum* feature should drive more usage of Windows Apps on the desktop. In these scenarios we as developers should think about keyboard shortcuts and how they can increase the productivity of their users.

So Windows 10 adds the `Windows.UI.Xaml.KeyAccelerator` class which provides a way to declare keyboard shortcuts for a `Windows.UI.Xaml.Controls.Page` in your xaml as you can see below.

``` xml
<Page.KeyAccelerators>
	<KeyAccelerator Key="N" KeyModifiers="Control" Pressed="OnNewProject" />
</Page.KeyAccelerators>
```

When that key combination is pressed (in this case Ctrl+N) then the `Pressed` event is invokes, it's handler looks like the following.

``` csharp
private void OnNewProject(KeyAccelerator key, object param)
{
	...  
}
```

## Conclusion

Key Accelerators are a nice simple feature to implement. I'd suggest as an exercise working through each of your views and determining the major commands for each and assign them a keyboard shortcut.
