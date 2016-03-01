---
layout: post
title: Xamarin acquisition thoughts
tags: microsoft xamarin
---

Last week it was [announced][gu] that Microsoft would be acquiring [Xamarin][xm], the popular toolset for building cross platform mobile apps with C#.

I presented a number of talks last year on [building cross platforms mobile apps with Xamarin][talks] that you can view online.

It's certainly an interesting piece of news, most people expected this to be announced at last year's Build (or even the year before). Back then there was a school of though that it was better to keep Xamarin as a close partner rather than acquiring them. The belief was that this would better enable Xamarin to work with other *"competitors*" such as Apple, Google and IBM.

I think now in 2016 it makes a lot of sense, the developer pick up of UWP is slow (IMO because there's currently on one platform with the desktop) and Microsoft need ways to drive more apps to the UWP platform.

Last year most of the announcements were around the application bridges, Astora, Centennial, Islandwood and Windsor (Android, Classic Windows, iOS and Web) as a way to bring more apps into the store (and hopefully the platform as well). Since the Astoria has been cancelled, we haven't see much of Centennial but Islandwood is looking really good.

This strategy didn't make themselves popular with a lot of Windows developers who thought these technologies would render them obsolete (a line of thought I didn't wholly agree with).

Microsoft appears to be positioning themselves quite nicely in the cross platform cloud and developer tools space (I think it's telling that announcement was on Scott Guthrie's blog). By having a tool-set where it isn't about migrating old apps but creating new cross platform apps I think they can better push Windows as a third platform as well as Android and iOS.

My thoughts on what may change over the next year or so.

- Xamarin subscription to be rolled into MSDN.
- Test Cloud to be part of Azure.
- Some people think Xamarin Studio may merge with Visual Studio Code, I doubt that. We may see the end of Xamarin Studio with efforts focussed on Visual Studio.
- Android Player to be deprecated in favour of Visual Studio Android Emulator.
- Hopefully better support for the Windows platforms in Xamarin.Forms, the default renderers are terrible.
- Xamarin Insights to be merged with Application Insights / HockeyApp.

Looking forward to what will be announced at Build, sadly will be missing this year though.

[gu]: https://weblogs.asp.net/scottgu/welcoming-the-xamarin-team-to-microsoft
[xm]: https://xamarin.com/
[talks]: http://compiledexperience.com/blog/posts/latest-talks