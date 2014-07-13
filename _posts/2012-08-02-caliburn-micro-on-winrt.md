---
layout: post
title: Caliburn Micro on WinRT
tags: csharp caliburn.micro windows-apps
---

Over the past week at [Marker Metro][mm] I’ve had the chance to spend some time porting [Caliburn Micro][cm] to WinRT. So I’m very proud to announce the release of our first experimental version. 

There’s still some bugs to work out and test but I’ve managed to integrate some awesome extensions into Windows 8 such as implementing the Share charm simply by having your view model implementing the interface ISupportSharing as well as support for the Settings charm.

There’s still lots of work, and I’m proud that the core changes are already incorporated in the [main project][source] but I’d also suggest checking out [our fork][fork] that includes the Windows 8 extensions and samples on how they work.

[mm]: http://markermetro.com
[cm]: http://caliburnmicro.codeplex.com/
[source]: http://caliburnmicro.codeplex.com/SourceControl/list/changesets
[fork]: http://caliburnmicro.codeplex.com/SourceControl/network/forks/marker_metro/CaliburnMicroWinRT
