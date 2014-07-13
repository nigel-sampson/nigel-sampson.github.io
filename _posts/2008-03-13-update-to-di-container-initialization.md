---
layout: post
title: Update to DI Container initialization
tags: csharp
---

Since I wrote the earlier code in &quot;[Using LINQ to initialize DI Containers](/blog/posts/update-to-di-container-initialization/)&quot;
I&#39;ve cleaned up the LINQ query to incorporate the assemblies loop and
also to use the First extension method to make things a bit more
readable.

``` csharp
var services = from a in assemblies
               from t in Assembly.Load(a).GetTypes()
               where t.GetInterfaces().Length > 0 && t.IsSubclassOf(typeof(ServiceBase))
               let i = t.GetInterfaces().First()
               where !Container.Kernel.HasComponent(i)
               select new
               {
                   Interface = i.IsGenericType ? i.GetGenericTypeDefinition() : i,
                   Component = t
               };
 
foreach(var service in services)
{
    Container.AddComponent(service.Interface.FullName, service.Interface, service.Component);
}
```

