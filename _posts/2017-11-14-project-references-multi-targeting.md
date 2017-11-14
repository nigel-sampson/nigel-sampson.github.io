---
layout: post
title: Project references to multi-targeted projects
tags: csharp xamarin
---

During the process of moving Caliburn.Micro to .NET Standard and the new [multi-targeting project format][mt] I've encountered a number of issues in the tooling around intellisense and builds. This isn't surprising given the relative newness of this approach, but I thought I'd share some of the issues over the next few weeks to help you out.

One of the first things I did was move `Caliburn.Micro.Platform` from a number of projects (around five I believe) in the same folder (one for each platform) to the new "SDK style" project format which allows multiple outputs based on a series of target frameworks (rather than the normal singular framework).

This worked out fine, but the unit tests in the solution started failing with compilation errors where certain classes were missing. In this case it was classes that aren't present in the Xamarin.Forms platform. This unit test project was a .NET 4.5 project and was clearly picking the wrong output of the multi-targeted project. Instead of picking the "best" platform of .NET 4.5, it was picking the widest in .NET Standard 1.4.

This is a known issue and you can see it being discussed on the GitHub repository for the new project system under ["P2P refs choose first tfm in multi-targting reference, not closest one in legacy project system"][bug]. 

If you've read the above issue you'll notice the way to solve this is to shift the other project to the new poject system. Once this is done the new project will pick the correct project output.

As I run into more problems during this port (hopefully not too many) I'll post them up here.

[bug]: https://github.com/dotnet/project-system/issues/1162
[mt]: https://caliburnmicro.com/announcements/net-standard