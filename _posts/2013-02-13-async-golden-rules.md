---
layout: post
title: Simple golden rules for async / await
tags: csharp windows-apps windows-phone
---

The async / await features in C# are incredibly easy to use but there’s a number of pitfalls than can be trap the unwary. The following are my golden rules when building any app (usually Windows 8 but now also Windows Phone 8) using the async / await features.

**Always await your async operations**, sometimes it’s easy to not await an async operation if you don’t need the result. By not awaiting the operation it becomes “fire and forget”, however this also means that any exceptions in the operation are swallowed and the operation could fail silently. I make sure all async operations are properly awaited so I can keep track of any errors in the system. If you need to make the operation “fire and forget” there are patterns you can use (I’ll go into these in a later post) to combine operations so you’re not holding other activity up.

**Be careful around async void**, there’s already advice on the internet to [avoid async void][void] methods as much as possible, but they have a habit of creeping in almost everywhere. In Windows 8 an exception that leaves an async void method will crash the process while also avoiding the Unhandled Exception handler (this does appear different in Windows Phone 8). Unfortunately you can’t easily get away from then, given that a lot of your async activity is driven from event handlers which will be async void. My golden rule is to **have a strategy in error handling to make sure all errors are handled in every async void**. I discuss one possible strategy in my [Tech Ed talk on Windows 8 app development][teched].

**Be very careful around async lambdas**, it can be very easy to use the async / await keywords inside lambdas, what is not easily apparent is the signature of the lambda. The same async lambda can be either an Action or a Func<Task> depending where you pass it, if it’s Action then you’re suddenly writing yourself an async void and the above advice applies, it also can’t be awaited with random effects. Phil Haack easily demonstrates what can happen if you [get it wrong][haack].

**Check that you really need to await the result**, often async methods have the following steps, use method arguments to create an async method call and return the result of the call. Much like the following:

``` csharp
private async Task<Repository> GetDefaultRepository()
{
    var id = GetDefaultRepositoryId();
 
    return await LoadRepository(id);
}
```

While there’s nothing strictly wrong with this code it does add some needless complexity and doesn’t perform as well as it should. The C# compiler creates a state machine internally to handle the re-entry into the method as well as dispatching the continuation correctly. This adds a slight performance penalty to every async method, not a huge one, but it can add up. The goal should be then to remove any needlessly async methods. All we need to do is remove both the async and await keyword methods, this means the Task of the underlying async operation is simply returns to the calling method to be await (or passed further back). In larger object hierarchies this can save about three or four methods from being async just because they call an async method.

``` csharp
private Task<Repository> GetDefaultRepository()
{
    var id = GetDefaultRepositoryId();
 
    return LoadRepository(id);
}
```

So these are my golden rules for using async / await. It’s not a complete guide but some things to think about and watch out for in your code base.

[void]: http://www.jaylee.org/post/2012/07/08/c-sharp-async-tips-and-tricks-part-2-async-void.aspx
[teched]: http://channel9.msdn.com/Events/TechEd/NewZealand/TechEd-New-Zealand-2012/APP301
[haack]: http://channel9.msdn.com/Events/TechEd/NewZealand/TechEd-New-Zealand-2012/APP301
