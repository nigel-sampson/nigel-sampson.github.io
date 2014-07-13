---
layout: post
title: Some thoughts about custom Uri Protocols
tags: windows-phone windows-apps
---

I've seen a lot of content around lately discussing the use of [custom uri protocols in your Windows Tablet and Phone apps][uri]. They're a great way to have other apps launch your app in a deep link kind of way, however I don't think developers are really using them to their full potential.

[Matt Lacey][matt] has a great post listing some of the 300 or so apps that implement custom uri protocols and their protocol name. Looking through the list a couple of things jump out at me.

The first is that there's no documentation about the protocol itself, even with the protocol name from the list I have no real idea about how to use it. Some developers like [Lazyworm Apps][lw] have [documented their metrotube protocol][mt] but the amount that have are few and far between. There needs to be a central resource for this list including documentation.

My second observation is that almost all the protocols are completely unique to that app, usually derived from the app name.  This means that any other app is limited to launch that and only that app. The end user also doesn't have a choice of which app they want to use to handle that protocol.

This problem is caused by a lack of global protocols besides the usual suspects of http, mailto etc. There are no established protocols for things like "watch a youtube video" leaving all the protocols as their own little islands.

What I'd love to see is the whole app protocol thing expanded so we can have a rich ecosystem of apps plugging into to fill different parts of a workflow, probably not going to happen but a guy can dream right?

[uri]: http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206987(v=vs.105).aspx
[matt]: http://blog.mrlacey.co.uk/2014/03/325-windows-phone-apps-you-can-launch.html
[lw]: http://lazywormapps.com/
[mt]: http://lazywormapps.com/metrotube-uri-schema.html

