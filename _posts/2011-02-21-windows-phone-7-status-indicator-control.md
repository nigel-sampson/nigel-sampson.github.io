---
layout: post
title: Windows Phone 7 Status Indicator Control
tags: csharp windows-phone
---

A lot of the built in applications have really subtle transitions and effects that really add polish to an existing application. In the process of building a new application I've been working on mimicking those controls. The first I'm releasing is the Status Indicator most commonly seen in the email application.

<img src="/content/images/posts/status.png" />

It's a fairly simple control that has two properties Text and InProgress controlling both the displayed status and whether the progress bar should be displayed. The control exposes two methods Display and Clear to easily set the properties of the control as well as manage state for nice animated transitions between the two.

``` xml
<controls:StatusIndicator x:Name="Status" />
```

``` csharp
private void OnDisplay(object sender, RoutedEventArgs e)
{
    Status.Display("Loading...", true);
}
 
private void OnClear(object sender, RoutedEventArgs e)
{
    Status.Clear();
}
```

One thing that frustrates me when dealing some out of the box controls is that there's often no nice way to interact with the control when using some sort of UI separation pattern such as MVVM. To deal with this the control exposes a Source property, you can then have a property on your view model that implements the IStatusSource interface and will allow you to determine the properties and state of the control from your view model. It's all very unit testable as well!

``` xml
<controls:StatusIndicator Source="{Binding Status}" />
```

``` csharp
public class StatusViewModel
{
    public StatusViewModel()
    {
        Status = new StatusSource();
    }
 
    public IStatusSource Status
    {
        get; set;
    }
 
    public void Display()
    {
        Status.Display("Updating", true);
    }
 
    public void Finish()
    {
        Status.Display(String.Format("Updated {0:d}", DateTime.Now), false);
    }
}
```


Of course here's the [source code](/content/downloads/compiledexperience.phone.toolkit.zip). 
