---
layout: post
title: Storyboards in Caliburn Micro
tags: csharp caliburn.micro wpf silverlight windows-phone
---

In my [previous post](http://compiledexperience.com/blog/posts/visual-state-manager-and-coroutines) I talked about the benefits of using co-routines in Caliburn Micro to ease any interactions with the View from the View Model. In that case it was the use of the Visual State Manager; in this post we’ll cover managing storyboards and animation.

We’ll use code from an [older post](http://compiledexperience.com/blog/posts/executing-code-on-animation-completion) around how to create one off event handlers. What I’ve done is encapsulate that logic into an extension method ToObservable.

``` csharp
public static IObservable<IEvent<EventArgs>> ToObservable(this Storyboard storyboard)
{
    if(storyboard == null)
        throw new ArgumentNullException("storyboard");

    return Observable.FromEvent((EventHandler<EventArgs> e) => new EventHandler(e),
                                e => storyboard.Completed += e,
                                e => storyboard.Completed -= e);
}
```

In the BeginStoryboardResult we verify the view is a FrameworkElement (and therefore can contain Resources). We then load the Storyboard from the Resources collection. Using the extension method we wire the Completed event of the Storyboard to the completion of the IResult.

``` csharp
public class BeginStoryboardResult : ResultBase
{
    private readonly string storyboardName;

    public BeginStoryboardResult(string storyboardName)
    {
        this.storyboardName = storyboardName;
    }

    public string StoryboardName
    {
        get { return storyboardName; }
    }

    public override void Execute(ActionExecutionContext context)
    {
        if(!(context.View is FrameworkElement))
            throw new InvalidOperationException("View must be a framework element to use BeginStoryboardResult");

        var view = (FrameworkElement)context.View;
        
        if(!view.Resources.Contains(StoryboardName) || !(view.Resources[StoryboardName] is Storyboard))
            throw new InvalidOperationException(String.Format("View doesn't the contain a Storyboard with the key {0} as a resource", StoryboardName));

        var storyboard = (Storyboard)view.Resources[StoryboardName];

        storyboard.ToObservable().Take(1)
            .Subscribe(e => OnCompleted());

        storyboard.Begin();
    }
}
```

After that it’s pretty much just starting the actual storyboard.

Again one of the main benefits of result an IResult like this is we still maintain separation between the view model and the view, we can now create unit tests that test how the view model plays storyboards without requiring the actual storyboard.

