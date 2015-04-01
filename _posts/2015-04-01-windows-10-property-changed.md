---
layout: post
title: What's new in Windows 10 - Property Changed Notifications
tags: windows-apps windows-phone  
---

Microsoft have finally started to show off what's available in Windows 10 for us app developers. Over the next few days we can start to take a look at some of these changes and what it can mean. Today we'll look at `DependencyProperty` changed notifications.

## Disclaimer

This was written using the initial release of the Windows 10 Developer Tools running on Windows 10 build 10041 and things will mostly change before final release. I'll try to come back and keep these posts up to date as time goes on.

## Property Changed Notifications

Most controls in Windows UAP expose events when significant changes to important properties are made. `TextBox` for instance exposes a `TextChanged` event. The trouble is that not every change is exposed that way and when they are it's through custom events, there's no reliable way to see property changed notifications across the board in a consistent way.

Thankfully that's changed in Windows 10 with `DependencyObject.RegisterPropertyChangedCallback`. This method allows you to register a delegate that's invoked whenever the given `DependencyProperty` changes. This means that even if a control doesn't expose an event for the property you care about there's still a way to do it.

Also because the signature of the delegate takes the `DependencyObject` and the `DependencyProperty` means that we can use this callback to register for multiple objects and properties.

In our example we want to register when the `IsReadOnly` property of a `TextBox` changes.

``` csharp
Username.RegisterPropertyChangedCallback(TextBox.IsReadOnlyProperty, (s, p) =>
{
	Debug.WriteLine("Is Read Only Changed: {0}", Entry.IsEnabled);
});
```

## Conclusion

Being able to subscribe to any dependency property for chnage notifications is one of those foundational features that for 95% of your projects you won't need it, but when you do you'll be incredibly thankful it's there.

I already am.