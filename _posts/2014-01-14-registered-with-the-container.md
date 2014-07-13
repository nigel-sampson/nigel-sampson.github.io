---
layout: post
title: Have you registered it with the container?
tags: csharp caliburn.micro windows-apps windows-phone silverlight wpf
---

These days there’s a plethora of Inversion of Control (IoC) containers in the .NET ecosystem, from classics such as [Castle Windsor][castle] and [Autofac][autofac] to the smaller ones that seem to ship with every MVVM framework. They all have their own idiosyncrasies in their behavior that really let you find one that best suits the way you code.

One behavior that’s a clear separator is how each container deals with a request that can’t be completed. This could be for a number of reasons, the type or interface isn't registered with the container, lack of public constructor etc. Some containers will throw an exception, others return null, and others like [MicroIoC][microioc] do their best to deal with it.

The SimpleContainer / WinRT Container in [Caliburn.Micro][caliburn] is one that returns null if a request for a type can’t be completed and will often the failing silently. Usually this manifests when you've just created a new view / view model pair, haven’t registered the view model into the container and you’re left wondering why your view model isn't working. For a while at the [Marker Metro][mm] offices we had a scoreboard the amount of times we’d forgotten and wasted time hunting for the problem (I can't reveal the leader).

One way to solve this is to use a container like [MicroIoC][microioc] that when a request for a concrete type (not abstract or an interface) that hasn’t been registered it will register that type automatically (under a Per Request lifecycle).

Depending on how long it’s been since I've run into the issue I’ll have different opinions on this behavior, this will certainly solve the aforementioned problem, but I often find it can lead to laziness in the registering of things other than view models and if any are statefull (such as a caching layer) you can run into some transparent bugs.

Another way to at least help you catch these errors during development time by modifying the GetInstance overload in Caliburn.Micro’s Bootstrapper / CaliburnApplication to something like as follows:

``` csharp
protected override object GetInstance(Type service, string key)
{
    var instance = container.GetInstance(service, key);
 
    if (instance == null)
        throw new InvalidOperationException(String.Format("Could not resolve type {0}", service));
 
    return instance;
}
```


While this will still fail silently as the exception is caught and handled within Caliburn.Micro, if you have a debugger attached you catch the exception and save yourself some precious time.

[castle]: http://www.castleproject.org/projects/windsor/
[autofac]: https://code.google.com/p/autofac/
[microioc]:https://github.com/PeteGoo/MicroIoc/
[caliburn]: https://microioc.codeplex.com/
[mm]: http://www.markermetro.com/
