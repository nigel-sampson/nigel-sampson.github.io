---
layout: post
title: Resource&#58; Logo Sizes for Windows 8
tags: windows-apps
---

I was all set to write a post on providing the scaled sizes (100%, 140%, 180%) for all the required images for a Windows 8 app (please don't forget about your [store logo][storelogo].

But while looking around I found "[Choosing your app images]()" had been updated in MSDN to include all the sizes. Which is great because I always forget them.

Once thing to note is that if you are providing scaled versions then your appxmanifest should refer to the image name without the scale suffix, even though that file doesn't technically exist. For example if you have StoreLogo.scale-100.png, StoreLogo.scale-140.png and StoreLogo.scale-180.png. Then the appxmanifest should refer to StoreLogo.png.

[storelogo]: http://compiledexperience.com/blog/posts/dont-forget-your-store-logo)
[msdn]: http://msdn.microsoft.com/en-us/library/windows/apps/hh846296.aspx
