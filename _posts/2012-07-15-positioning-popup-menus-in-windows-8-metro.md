---
layout: post
title: Positioning Popup Menus in Windows 8 Apps
tags: csharp windows-apps
---

Often when building Windows 8 apps you'll want to provide a menu of options from the app bar, this may be a list of different ways to sort or filter the data. For these simple menus WinRT provides the [PopupMenu control][msdn] (for more complicated popup user interfaces I recommend the Flyout control from the [Callisto library][callisto]), these menus are very easy to create by creating UI Commands and appending them to the menus Commands collection.

![A popup menu][popup]

Once you have the menu created you need a point to display the menu, there a couple of different ways to achieve this but I've found the best way is to use the following approach. First we use TransformToVisual to create a transform, by passing the Page as the parameter the result is the screen coordinates for the button. Then we create point object that represents the offset from the button, in this example we offset to the right and to the top by 45 and -10 pixels respectively. We then use the transform to create a point that’s the position of the button and the offset; we can then call ShowAsync with our computed point.

``` csharp
var popupMenu = new PopupMenu();

popupMenu.Commands.Add(new UICommand("Watched", async h => await ViewModel.FilterRepositories(RepositoryFilter.Watched)));

popupMenu.Commands.Add(new UICommand("Owned", async h => await ViewModel.FilterRepositories(RepositoryFilter.Owned)));

var button = (Button)sender;

var transform = button.TransformToVisual(this);
var point = transform.TransformPoint(new Point(45, -10));

await popupMenu.ShowAsync(point);
```

[popup]: /content/images/posts/popup-menu.png 
[MSDN]: http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.popups.popupmenu.aspx
[callisto]: https://github.com/timheuer/callisto


