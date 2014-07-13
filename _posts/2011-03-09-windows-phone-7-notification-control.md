---
layout: post
title: Windows Phone 7 Notification Control
tags: csharp windows-phone
---

One of the controls I felt was missing out of the box for the Windows Phone 7 was a Notification control similar to how Toast notifications are displayed. As part of an application I'm working on I need a control like this with some of the animation effects the toast has.

<img src="/content/images/posts/notification.png">

On a side note the [Coding4Fun Toolkit](http://coding4fun.codeplex.com/) has a more fully featured control but lacks good view model integration (to be fair 90% of controls do).

It has three properties, Title and Text (I'll most likely be adding an icon at a later date) are fairly self explanatory, the third OnDimiss is a callback when the notification is dismissed by a tap.

``` xml
<controls:Notification x:Name="Notification" />
```

From the code behind it displayed with the following calls.

``` csharp
private void OnDisplay(object sender, RoutedEventArgs e)
{
    Notification.Display("Title", "Text", () =>
    {
        MessageBox.Show("Dismissed", "The notification control has been dimissed", MessageBoxButton.OK);
    });
}
 
private void OnDismiss(object sender, RoutedEventArgs e)
{
    Notification.Dismiss();
}
```

From a view model we can expose a unit testable notification source and bind that to the control.

``` csharp
public class NotificationViewModel
{
    public NotificationViewModel()
    {
        Notifications = new NotificationSource();
    }
 
    public INotificationSource Notifications
    {
        get;
        set;
    }
 
    public void Display()
    {
        Notifications.Display("Security", "Tap to authenticate the application", () =>
            {
                MessageBox.Show("Authenticated", "The application has been authenticated against the server", MessageBoxButton.OK);
            });
    }
}
```

The [source code here](/content/downloads/compiledexperience.phone.toolkit.zip) contains the code for this notification control as well as the [Status Indicator control](/blog/posts/windows-phone-7-status-indicator-control).
