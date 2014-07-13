---
layout: post
title: My essential tools and resources for Windows 8 & Windows Phone development
tags: windows-phone windows-apps
---

A lot of people have recently asked about the libraries, tools and resources I use to build Windows 8 and Windows Phone applications so I’d like to cover off as many as I can. It’s not an exhaustive list but I wouldn’t approach an app without almost all of these being involved.

###Libraries
[Caliburn Micro][cm] Windows Phone, Windows 8, unless the app is a throw away prototype I always use Caliburn Micro as my MVVM framework of choice (I may be a little biased these days though). The conventions help in bringing down the amount of plumbing code you need and it can do as little or as much as you want with a huge number of configuration and extension points.

[Akavache][akavache] Windows Phone, Windows 8, a hidden gem from the guys and gals at GitHub. Every app should use some sort of local caching for performance reasons and Akavache makes this incredibly easy. For Windows 8 I’d recommend the SQLite backend.

[Json.Net][json] Windows Phone, Windows 8, so many API’s use JSON as the transfer format and Json.Net is one of the easiest libraries to use.  Developed by fellow kiwi James Newton King.

[Callisto][callisto] Windows 8, a UI library from Tim Heuer that helps fills the gaps in the WinRT xaml controls collection. The Flyout and Menu controls are incredibly useful.

[Telerik][telerik] / [Syncfusion][syncfusion] / [Mindscape][mindscape] Windows Phone, Windows 8, all the xaml platforms have third party control vendors. For Windows Phone I couldn’t go past Telerik, their portfolio of controls covers almost all of my requirements, one of the best ways to get these controls is to join the [Nokia Developer program][nokia].  On the Windows 8 side the pickings are a little slimmer thanks to the newness of the platform. I’d recommend looking over the libraries of Telerik, Syncfusion and Mindscape to find the set that best suits your needs.

###Tools
[XamlSpy][xaml] Windows Phone and Windows 8, much like the DOM inspection tools present in all browsers these days XamlSpy lets you inspect the visuals of your xaml app at run time. I find it useful for testing alignments by overlaying a grid and for discovering where some surprising margins and paddings come from (I’m looking at you FlipView). To help get your app over the final quality hurdles I wouldn’t go for anything else.

[Expression Design][design], while technically discontinued I’ve found Expression Design a nice tool for tweaking simple visual assets such as icons and logos. For people like me who find something like Photoshop a bit of overkill this may be useful.

[Inkscape][ink], one approach to get around the multiple image requirements for both Windows 8 and Windows Phone is to use vector paths in your app, these will scale to any size without losing detail or becoming pixelated. Inkscape is an svg tool that has a useful “Save to xaml” tool that comes in really handy.

###Resources
[Modern UI Icons][mui] Windows Phone and Windows 8, hands down the best iconography resource for all Modern UI (cough, Metro) apps. As every icon in png, xaml and design (for those of us who still use Expression Design) which is incredibly useful.

[Color Hexa][color], while I certainly don’t have a particularly advanced design skillset it’s useful to be able to find appropriate colours. Color Hexa provides all the information about a given colour you’ll ever need. I use it to find different shades of a colour and to find it’s compliment for use as an accent colour for instance.

While this isn't an exhaustive list I hope it gives you an idea of some of the tools, libraries and resources out there that you can use.

[cm]: https://github.com/BlueSpire/Caliburn.Micro
[akavache]: https://github.com/github/Akavache
[json]: https://json.codeplex.com/
[callisto]: https://github.com/timheuer/callisto
[telerik]: http://www.telerik.com/developer-productivity-tools.aspx
[syncfusion]: http://www.syncfusion.com/
[mindscape]: http://www.mindscapehq.com/
[nokia]: http://www.developer.nokia.com/Developer_Programs/
[xaml]: http://xamlspy.com/
[design]: http://www.microsoft.com/expression/eng/
[ink]: http://inkscape.org/
[mui]: http://modernuiicons.com/
[color]: http://www.colorhexa.com/
