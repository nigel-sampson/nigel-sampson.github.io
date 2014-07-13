---
layout: post
title: Using JQuery to embed Silverlight
tags: mvc silverlight
---

Embeding Silverlight xaps in a page can be a pain in the ass, it's very verbose if you try and do it manually. I'm already using JQuery on the Compiled Experience Website so anything where I can use JQuery to simply this would be great.

The best example I've found of doing this is on the [Silverlight Forums](http://forums.silverlight.net/forums/p/183327/419039.aspx) and is a great simple little plugin that lets you embed Silverlight xaps. I've modifed the example one slightly (mostly method names). Once you've included both JQuery and the [plugin js file](http://compiledexperience.com/content/scripts/jquery.silverlight.js) it's as simple as this to embed your Silverlight.

``` javascript
$(function () {
  $("#focus-panel").silverlight("../../clientbin/controls.focuspanel.xap", {
    width: 560,
    height: 300
  });
});
```

