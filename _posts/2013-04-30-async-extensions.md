---
layout: post
title: Some useful async extension methods
tags: csharp
---

Linq is awesome, async / await is awesome, but combining them can be a little awkward. None of the standard linq operations are written to deal with async lambdas, this often leads to writing the async operations in a more procedural style approach rather than the functional one that Linq uses. 

Here's a couple of useful extension methods I use, WhenAllAsync allows you to execute an aysnc method on a sequence of values. By using Task.WhenAll we can have the async method execute in parallel with each other. 

``` csharp
public static Task WhenAllAsync<T>(this IEnumerable<T> values, Func<T, Task> asyncAction)
{
    return Task.WhenAll(values.Select(asyncAction));
}
```

The usage is pretty self explanatory, the only odd bit is that because DeleteAsync returns an IAsyncOperation rather than Task we use AsTask to convert it (or if you want right a second extension method using IAsyncOperation).

``` csharp
var files = await GetFilesAsync();
 
await files.WhenAllAsync(f => f.DeleteAsync().AsTask());
```

SelectAsync is useful for transforming a collection of values to another type via an asynchronous method. It's much like WhenAllAsync except that we return the result of the operation. 

``` csharp
public static async Task<IEnumerable<TResult>> SelectAsync<TSource, TResult>(this IEnumerable<TSource> values, Func<TSource, Task<TResult>> asyncSelector)
{
    return await Task.WhenAll(values.Select(asyncSelector));
}
```

``` csharp
var images = await GetFilesAsync();
 
var streams = await files.SelectAsync(f => f.OpenAsync(FileAccessMode.Read).AsTask());
```

Hopefully these help you write more readable performant async code.
