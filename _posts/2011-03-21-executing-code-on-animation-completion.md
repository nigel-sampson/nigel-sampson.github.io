---
layout: post
title: Executing code on animation completion
tags: csharp silverlight wpf windows-phone
---

The eventing mechanisms in .NET are great for being notified when certain things occur, but often you don't care about every occurrence just the next one, really what you're wanting is a callback. The most common scenario I've encountered for this during my development on the Windows Phone 7 is wanting to execute code after an animation is complete and by extension when an animation part of a visual state is complete.

What I want to show is two different mechanisms to create one time event handlers, then how to build simple extension methods to make use of these to create callbacks and then also how to use it with animations and the visual state manager.

The first and easiest method is a self removing event handler, rather than simply adding an event handler to the event we first create the event handler as a separate variable initialised as null. We then initialise the handler as a lambda that does what you need to do and then takes advantage of closures to remove itself from the event.

This can seem quite complicated, what makes this work is that we're splitting what would usually be one statement (declaring the event handler and assigning to the event) into three, declaring the reference to the handler, declaring the handler itself and assigning to the event. By splitting the declaration of the variable and the initialisation means we can reference the variable in the handler itself.

``` csharp
public void AttachWithHandler()
{
    EventHandler handler = null;
 
    handler = (s, e) =>
    {
        MessageBox.Show("Completed!");
 
        storyboard.Completed -= handler;
    };
 
    storyboard.Completed += handler;
}
```

The alternate way is to use the new library from Microsoft called [Reactive Extensions](http://msdn.microsoft.com/en-us/devlabs/ee794896) this allows us to use a more declarative syntax, for one off event handlers the important method is the Take(1). What's cool about this approach is that you can handle multiple events easily over the first approach. We can also add filters and other features of the Reactive Extensions framework to our handler if required.

``` csharp
public void AttachWithRx()
{
    Observable.FromEvent((EventHandler<EventArgs> e) => new EventHandler(e),
        e => storyboard.Completed += e,
        e => storyboard.Completed -= e)
        .Take(1)
        .Subscribe(e => MessageBox.Show("Completed"));
}
```

So now we have our approaches lets build some extension methods to make this into a simple extension method, rather than caring about a full event handler all we really want is to be able to define a callback method and have that executed on event completion. To solve our storyboard completion animation problem we'll create an extension for storyboard that takes a callback and then uses the first approach to use it as a one off event handler.

``` csharp
public static void Begin(this Storyboard storyboard, Action callback)
{
    EventHandler handler = null;
 
    handler = (s, e) =>
    {
        callback();
        storyboard.Completed -= handler;
    };
 
    storyboard.Completed += handler;
 
    storyboard.Begin();
}
```

Here's how I'm using it in [To Do Today](/windows-phone/to-do).

``` csharp
private void OnDeleteAll(object sender, EventArgs e)
{
    AnimateDelete.Begin(ViewModel.DeleteAll);
}
```
