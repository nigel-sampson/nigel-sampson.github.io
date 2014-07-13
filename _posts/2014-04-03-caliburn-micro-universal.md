---
layout: post
title: Caliburn.Micro and Universal Apps
tags: caliburn.micro windows-apps windows-phone
---

There’s been a lot of announcements here at [Build][build] today, including the availability of the Windows Phone 8.1 SDK. This release brings a common Xaml UI framework between Windows Phone and Windows and some tooling support to enable you to easily share C# and Xaml between the two platforms called [Universal Windows apps][uni].

I’m also really pleased to say I’ve recently pushed the code to enable you to use [Caliburn.Micro][cm] on the new frameworks. It’s available on [GitHub][cm] and on [Nuget][nuget] under version 2.0.0-beta2, this version adds the new Windows Phone 8.1 platform to the Portable core assembly and appropriate Platform assembly.

Because the new Windows Phone platform is closely aligned with the Windows platform you’ll notice that we build the app like it’s a Windows 8.1 app. For CM this means a few changes from Windows Phone 8, such as using CaliburnApplication rather than Bootstrapper, but for the most part most of the tutorials for using CM on Windows 8.1 will apply to Windows Phone 8.1. 

If you still want to build using the Windows Phone 8 Silverlight platform then WP8 version is still available from the same package and works just as it did before.

To show off how powerful it is now the repository has a [sample Universal app][sample] where we share views and view models across both platforms, but still enable us to have custom views for shared view models. This coupled with Portable class libraries enables a variety of strategies to share code over many number of platforms. I’ll hopefully be covering a lot of these in the near future.

As we get to grips with some of the stuff still to be announced over the coming days there should be more to add to this.

On a related note I just want to thank [Marker Metro][mm], I wouldn’t have been able to get this release ready so quickly without their support.

[build]: http://www.buildwindows.com/
[cm]: https://github.com/BlueSpire/Caliburn.Micro
[mm]: http://www.markermetro.com
[nuget]: http://www.nuget.org/packages/Caliburn.Micro/2.0.0-beta2
[sample]: https://github.com/BlueSpire/Caliburn.Micro/tree/master/samples/CM.HelloUniversal
[uni]: http://blogs.windows.com/windows/b/buildingapps/archive/2014/04/02/windows-store-universal-windows-app-opportunity.aspx
