---
layout: post
title: Interesting uses of tuple deconstruction
tags: csharp
---

C# 7.0 brought us a new and interesting feature with tuple types and tuple literals. These coupled with tuple deconstruction let us create new syntax patterns and helper methods that hopefully results in more readable code.

One bug bear of mine was the way we ended up having to use `Task.WhenAll` when wanting to await mutiple tasks at the same time. If you didn't need the results of the `Task`'s then it wasn't too bad.

``` csharp
await Task.WhenAll(StartTaskOne(), StartTaskTwo());
```

but when we want the results of the tasks when end up with syntax that looks like

``` csharp
var count = GetCount();
var description = GetDescription();

await Task.WhenAll(count, description);

Console.WriteLine($"Count: {count.Result}, Description: {description.Result}");
```

What we can do is create an extension method that extends a `ValueTuple` of `Tasks`'s awaits them and results another tuple of the results.

``` csharp
public static class TaskExtensions
{
    public static async Task<ValueTuple<T1, T2>> WhenAll<T1, T2>(this ValueTuple<Task<T1>, Task<T2>> tasks)
    {
        await Task.WhenAll(tasks.Item1, tasks.Item2);

        return (tasks.Item1.Result, tasks.Item2.Result);
    }
}
```

When then end up with more readable

``` csharp
var (count, description) = await (GetCount(), GetDescription()).WhenAll();

Console.WriteLine($"Count: {count}, Description: {description}");
```

Sadly the above extension method only works with two tasks, but you can see how you'd write a three task version. As the C# language stands right now there's no way to build a generic version that would work with any amount of tasks (though I'd be very happy to be proven wrong here). I'd argue though if you need overloads of this method beyond four or five something has gone awfully wrong in your code base.