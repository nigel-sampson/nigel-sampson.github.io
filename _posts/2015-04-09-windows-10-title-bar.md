---
layout: post
title: What's new in Windows 10 - Title Bar
tags: windows-apps windows-phone
---

Microsoft have finally started to show off what's available in Windows 10 for us app developers. Over the next few days we can start to take a look at some of these changes and what it can mean. Today we'll look at what options we have to configure the new window Title Bar.

## Disclaimer

This was written using the initial release of the Windows 10 Developer Tools running on Windows 10 build 10041 and things will mostly change before final release. I'll try to come back and keep these posts up to date as time goes on.

## Title Bar

Now that our apps will work look and work like any standard desktop application we'll find them framed in a window with a title bar. By default the title bar will look like the image below with a grey bar. It will have the app name on the left and three standard windows buttons on the right.

<a href="/content/images/posts/default.png"><img width="600" src="/content/images/posts/default.png"/></a>

So what can we customise? Most things thankfully. The code below sets the text on the left to **&lt;app name&gt; - My Custom Title**. So it could be a good idea to update this each page of your app.

``` csharp
var applicationView = ApplicationView.GetForCurrentView();

applicationView.Title = "My Custom Title";
```

The most likely thing we're going to want to do is customise the colours. We can do that using the `TitleBar` property of `ApplicationView`.

``` csharp
var applicationView = ApplicationView.GetForCurrentView();
var titleBar = applicationView.TitleBar;

titleBar.BackgroundColor = (Color) Resources["SidebarBackgroundColor"];
titleBar.ForegroundColor = Colors.White;

titleBar.ButtonBackgroundColor = (Color) Resources["LightSidebarBackgroundColor"];
titleBar.ButtonForegroundColor = Colors.White;
```

The first two properties change the background the bar itself and the colour of the title text. The second two set the background and foreground of the buttons. There's actually a LOT of properties for the buttons covering their hover and pressed states. I'd recommend checking out the documentation on [MSDN](https://msdn.microsoft.com/en-ca/windows.ui.viewmanagement.applicationviewtitlebar).

The code above gives us

<a href="/content/images/posts/colours.png"><img width="600" src="/content/images/posts/colours.png"/></a>

A third option is to remove the title bar altogether for that *chrome-less* look. We can use the `ExtendViewIntoTitleBar` property.

``` csharp
var applicationView = ApplicationView.GetForCurrentView();
var titleBar = applicationView.TitleBar;

titleBar.ExtendViewIntoTitleBar = true;

titleBar.ButtonBackgroundColor = (Color) Resources["LightSidebarBackgroundColor"];
titleBar.ButtonForegroundColor = Colors.White;
```

This gives me that best result for my app in my opinion, but will vary with the style and content of each app.

<a href="/content/images/posts/extend.png"><img width="600" src="/content/images/posts/extend.png"/></a>

## Conclusion
Now that how apps are going to be in windows a lot of the time we need to pay more attention to things like the title bar.