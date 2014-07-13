---
layout: post
title: Scaling with a PathIcon
tags: windows-apps
---

The new [AppBarButton][abb] control in Windows 8.1 is incredibly useful, the ability to use symbols, gylphs, images or paths as an icon within the button makes it a lot easier to customise that in 8.0. There's just one slight problem. 

[Path Icon][pi] doesn't scale the Path geometry to fit within the button, it renders just as the Path geometry is specified which is kind of awkward. 

We could try and re-template PathIcon but that's probably overkill. The easiest thing to do is to rescale the Path, tools like [Inkscape][is] would be one way to do it. Thankfully Christian Mosers has created a really simple tool to transform Path geometry down to the appropriate scale. It's called [Geometry Transformer][gt] and is available on his [website][gt]. 

[abb]: http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.controls.appbarbutton.aspx
[pi]: http://msdn.microsoft.com/en-us/library/windows/apps/windows.ui.xaml.controls.pathicon.aspx
[is]: http://www.inkscape.org/en/
[gt]: http://wpftutorial.net/GeometryTransformer.html
