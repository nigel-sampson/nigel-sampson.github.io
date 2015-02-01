---
layout: post
title: Preventing zoom on WebView	
tags: windows-apps windows-phone  
---

In an upcoming version of [Hub Bug][hb] I'm using an embedded WebView to display Markdown content (yay!). This works fantastically in a desktop experience, touch however is a different experience.

Web has some peculiarities about it's layout, it won't resize to fit it's content instead it needs to be a fixed layout and to make use of it's built in scrolling for any content larger than it's current viewport.

We can work around this issue most of the time, however part of this scrolling comes zooming which means on touch devices the user can pinch zoom your content which usually isn't something you want with an embedded WebView.

So how do we disable this? Initially I experimented with the `viewport` with no success.

``` html
<meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0">
```

Thankfully [Marker Metro][mm]'s resident html/js expert [Damian Karzon][dk] came to the rescue with the following css.

``` css
html {
    -ms-content-zooming: none;
}
```

We now have embedded html content in the app that feels a part of the app and not that weird *floating on top*.

[hb]: http://compiledexperience.com/windows-apps/hub-bug/
[dk]: https://twitter.com/dkarzon
[mm]: http://www.markermetro.com/