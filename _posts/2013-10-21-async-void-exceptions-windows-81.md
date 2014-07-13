---
layout: post
title: Async Void, Exceptions and Windows 8.1
tags: csharp windows-apps
---

[Many people][async-tricks] including [myself][async-rules] have talked about the dangers of async void methods in your Windows 8 application. The fact that exceptions in these methods ultimately caused the application to crash was a big problem, realistically though trying to build a proper application with no async void methods is an exercise in futility. There are ways to mitigate the problem, from [replacing the Synchronization Context][sync-context] like we do at [Marker Metro][mm], [IL Weaving using Fody][fody] to plain exception handlers in every async void method.

Thankfully in appears that in Windows 8.1 some of these problems have been alleviated, you’ll notice now that exceptions throw in async void methods will now be passed to the Application.UnhandledException event. There if you set the event arguments Handled property to true you can recover your application from perhaps an unnecessary crash.

``` csharp
protected override void OnUnhandledException(object sender, UnhandledExceptionEventArgs e)
{
    e.Handled = true;
    LogException(e.Exception);
}
```

All in all I love these little updates to WinRT, here’s hoping we can find some more of them.

[async-tricks]: http://www.jaylee.org/post/2012/07/08/c-sharp-async-tips-and-tricks-part-2-async-void.aspx
[async-rules]: http://compiledexperience.com/blog/posts/async-golden-rules
[sync-context]: http://www.markermetro.com/2013/01/technical/handling-unhandled-exceptions-with-asyncawait-on-windows-8-and-windows-phone-8/
[mm]: http://www.markermetro.com/
[fody]: https://github.com/Fody/AsyncErrorHandler

