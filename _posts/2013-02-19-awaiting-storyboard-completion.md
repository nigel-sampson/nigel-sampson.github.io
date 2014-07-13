---
layout: post
title: Awaiting Storyboard completion
tags: csharp windows-phone windows-apps
---

Quite a while ago I wrote a post on two different mechanisms to how to [execute code on animation completion][previous], it’s a handy trick when you need to chain different animations and code together. Back then I used two different approaches, the first was a self-removing event handler and the second with [Reactive Extensions][rx]. With the coming of C# 5 and the async / await languages features it would make sense that we’d want to “await” an animation, in fact this approach makes for the most readable code for chaining these things together.

We still need to use one of the previous solutions to trigger our async code but we can now wrap it in such a way as to make it “await-able”.

``` csharp
await DeleteAnimation.BeginAsync();
```

The class that does all the work for us is [TaskCompletionSource][msdn], it’s a class that allows us to create tasks behind the scenes and control whether they are completed, cancelled or failed. With this task you can build up some really handy mechanisms to improve the readability of your code. I brief discussion of the class is available at [The Nature of TaskCompletionSource<TResult>][tcs] that I highly recommend.

``` csharp
public static Task BeginAsync(this Storyboard storyboard)
{
    var taskSource = new TaskCompletionSource<object>();
 
    EventHandler<object> completed = null;
 
    completed += (s, e) =>
    {
        storyboard.Completed -= completed;
 
        taskSource.SetResult(null);
    };
 
    storyboard.Completed += completed;
 
    storyboard.Begin();
 
    return taskSource.Task;
}
```

For our purposes we’ll create a TaskCompletionSource&lt;object&gt; (there’s no non-generic version so we’ll just use object). We then set up our self-removing event handler as in the previous article, but instead of triggering some sort of callback we call SetResult on the completion source. This marks the underlying task as completed with a result of null (which we’ll promptly ignore since we only care that it’s completed). We then begin the storyboard and return the underlying task to the calling method.

The execution flow runs something like this:

 1. BeginAsync is called the event handler is wired to the storyboard.
 2. The storyboard is started.
 3. The Task is returned to the calling method.
 4. The Task is awaited by the calling method.
 5. The storyboard finishes its animation and triggers the event handler.
 6. The event handler sets the result of the Task.
 7. The calling method finishes awaiting and continues.

Hopefully you can see from this every simple pattern how you can use TaskCompletionSource to make other potentially async parts of your code base simpler.

[previous]: http://compiledexperience.com/blog/posts/executing-code-on-animation-completion
[rx]: http://msdn.microsoft.com/en-us/data/gg577609
[tcs]: http://blogs.msdn.com/b/pfxteam/archive/2009/06/02/9685804.aspx
[msdn]: http://msdn.microsoft.com/en-us/library/dd449174.aspx
