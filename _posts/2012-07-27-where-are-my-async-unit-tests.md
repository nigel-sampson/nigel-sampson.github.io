---
layout: post
title: Where are my async unit tests?
tags: csharp windows-apps
---

I love the new unit test UI in Visual Studio 2012, especially the support for asynchronous unit tests.  However simply adding the async keyword isn’t quite enough, Visual Studio won’t detect unit tests with the method signature “public async void”. 

``` csharp
[TestMethod]
public async void IsCacheValidAsyncWithExpiredItemReturnsFalse()
```

For Visual Studio to detect asynchronous unit tests they need to return System.Threading.Tasks.Task, it’s this type that lets the unit test framework catch exceptions in the asynchronous method, without it any of the assertion failures would be swallowed.

``` csharp
[TestMethod]
public async Task IsCacheValidAsyncWithExpiredItemReturnsFalse()
```

Happy testing.


