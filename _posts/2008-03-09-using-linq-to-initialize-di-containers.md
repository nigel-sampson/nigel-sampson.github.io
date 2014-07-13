---
layout: post
title: Using LINQ to initialize DI Containers
tags: csharp
---

Recently there have been some discussion on how to help initialize dependency injection containers. Ayende call this &quot;[Setting up Zero Fiction Projects](http://www.ayende.com/Blog/archive/2008/02/09/Setting-Up-Zero-Friction-Projects--Dependency-Injection.aspx)&quot;
and uses the Boo language to create a DSL (Domain Specific Language) to
specify components to be registered into the container. From [hammet](http://hammett.castleproject.org/?p=253)
comes news around a fluent interface for adding components to Castle
Windsor. The essence of both is to find all types in a given assembly
that either implement a certain a certain interface or are a subclass
of a layer supertype (I tend to have a ServiceBase that contains things
Log and so on). We can use LINQ to achieve something similar to both
the above approaches.

``` csharp
public static void RegisterControllers(params string[] assemblies)
{
    foreach(string assembly in assemblies)
    {
        IEnumerable<Type> controllers = from t in Assembly.Load(assembly).GetTypes()
                                        where t.Implements<IController>()
                                        select t;
 
        foreach(Type controller in controllers)
        {
            Container.AddComponentWithLifestyle(controller.FullName, controller, LifestyleType.Transient);
        }
    }
}
```

The above code iterates a list of assembly names and finds all types
that implement the ASP.NET MVC IController interface. To increase the
readability of the LINQ query I&#39;ve used an extension method listed
below.

``` csharp
public static bool Implements<T>(this Type type)
{
    return typeof(T).IsAssignableFrom(type);
}
```

The below code locates the first interface for the service and uses
that as the service to register the type. Depending on how you create
your types this may or may not be a good method. Using the Repository
class I&#39;ve described before doesn&#39;t allow this approach as it is the
last interface that becomes the important one, however I&#39;ve used this
code to show some similar functionality as the examples above. Hope
this helps someone.

``` csharp
public static void RegisterServices(params string[] assemblies)
{
    foreach(string assembly in assemblies)
    {
        var services = from t in Assembly.Load(assembly).GetTypes()
                       where t.GetInterfaces().Length > 0 && t.IsSubclassOf(typeof(ServiceBase))
                       let i = t.GetInterfaces()[0]
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
    }
}
```

