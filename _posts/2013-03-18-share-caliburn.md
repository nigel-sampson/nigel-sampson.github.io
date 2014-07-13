---
layout: post
title: WinRT Sharing with Caliburn Micro
tags: csharp caliburn.micro windows-apps
---

With the [version 1.5 release of Caliburn Micro][release] we now have seamless support for having your WinRT act as a Share source. Acting as a Share source allows the user to send data from your app to other WinRT apps, it’s great in that you don’t need to build “export to …” functionality into your apps because other apps can provide the functionality for you.  For instance [Hub Bug][hubbug] acts as are Share target so people can quickly create GitHub issues from other applications.


To set up your app the first thing we need to do is enable the sharing functionality. We do this with the method RegisterSharingService on WinRTContainer, typically in the Configure method of App.xaml.cs. Now if you run the app and activate the share charm (I prefer the shortcut Win+H) you’ll see the text “There’s nothing to share right now.”

``` csharp
container.RegisterSharingService();
```

The next step is to provide hooks in the different sections you want to support sharing. To do this we use the interface **ISupportSharing**. 

``` csharp
public class ShareSourceViewModel : Screen, ISupportSharing
```

When you activate the share charm Caliburn Micro will look at the current view model to see if it implements this interface. If it does it will call **OnShareRequested** which you can then populate the request with the data the current view model represents.

``` csharp
public void OnShareRequested(DataRequest dataRequest)
{
    dataRequest.Data.Properties.Title = "Share Source Example";
    dataRequest.Data.Properties.Description = "An example of sharing data from a view model";
 
    dataRequest.Data.SetUri(new Uri("http://www.compiledexperience.com"));
}
```

If you need to use async methods within OnShareRequested then you can use the deferral support on DataRequest like so.

``` csharp
public async void OnShareRequested(DataRequest dataRequest)
{
    var deferral = dataRequest.GetDeferral();
 
    dataRequest.Data.Properties.Title = "Share Source Example";
    dataRequest.Data.Properties.Description = "An example of sharing data from a view model";
 
    var text = await GetDataAsync();
 
    dataRequest.Data.SetText(text);
 
    deferral.Complete();
}
```

If you implement the interface but the current view model has nothing to share (nothing selected for example) then simply return from the method without setting any data on the request.

[release]: http://devlicio.us/blogs/rob_eisenberg/archive/2013/03/18/durandal-1-2-0-and-caliburn-micro-1-5-0-released.aspx
[hubbug]: http://compiledexperience.com/windows-apps/hub-bug
