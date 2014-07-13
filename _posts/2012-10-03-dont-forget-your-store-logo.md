---
layout: post
title: Don’t forget your Store Logo
tags: windows-apps
---

If you’ve been browsing the new Windows 8 Store you may notice every once in a while an app with an icon that looks like the listing on the left.

![Example of default logo][store]

This isn’t a bug but an indication that the developer hasn’t included a Store Logo with their app. To be honest I’m surprised this doesn’t trigger a Store submission error but for whatever reason some apps are getting through with the default Store Logo. The reason why a reasonable amount of developers are missing it is that while the setting lives in the Package.appxmanifest like the other logos it’s under the Packaging tab and not the Application UI tab. The requirement is that this image should be 50 x 50 pixels.

![Example of changing default logo][manifest]

[store]: /content/images/posts/store-logo.png
[manifest]: /content/images/posts/manifest-logo.png

