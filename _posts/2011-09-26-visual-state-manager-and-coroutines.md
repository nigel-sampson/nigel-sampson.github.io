---
layout: post
title: Visual State Manager and Coroutines
tags: csharp caliburn.micro silverlight windows-phone
---

Well it's been quite a while since I've posted anything having spent the last five weeks in Canada getting married and St Lucia on our honeymoon. As usual a lot happens while you're away, with Mango getting closer to release (and the marketplace opening for Mango apps), the unveiling of Windows 8 at Build and other smaller news. With all this there's been a lot to catch up but after five weeks away from development I've been itching to get back to it. I’m currently working on Mango updates for [Left to Spend](http://compiledexperience.com/windows-phone-7/left-to-spend) and [To Do Today](http://compiledexperience.com/windows-phone-7/to-do), I’m taking the opportunity to rebuild bits of To Do Today with some of the techniques I’ve learnt in the last year so I’ll hopefully be discussing some of those as I go.

One of the cool and often over looked features in [Caliburn Micro](http://caliburnmicro.codeplex.com/) is the IResult interface and its use in Coroutines, there's some [excellent documentation](http://caliburnmicro.codeplex.com/wikipage?title=IResult%20and%20Coroutines&referringTitle=Documentation) on the Caliburn Micro website so I won't go into a lot of detail regarding how they work. One of the aspects I really like (besides helping to simplify asynchronous code) is that they better enable View, View Model separation even when you need to interact directly with the View.

A great example of this is the Visual State Manager, the visual state of a screen is something that there's a definite use case to control from the View Model as it’s a smooth way to have animated states within the page. Previously my approach has been to expose a string "State" property on the View Model and bind this to a custom attached dependency property. This works but I feel it has a few weaknesses, a page can be in multiple states at any one time due to Visual State Groups (the page will be in a state for each group). This isn't represented well by the State property and in reality really hides that fact. It also doesn’t control the need to turn on and off state transitions.

Because the ActionExecutionContext passed to the IResult has a reference to the View we can manipulate the Visual State Manager directly in the IResult without worrying about bindings. We’ve also preserved the distinction between the View and View Model so we can unit test our View Model changes the state without execution of the IResult.
So what does the code look like, pretty simple really, I have a ResultBase class to handle some of the plumbing in IResult. From this class I create my VisualStateResult that takes the appropriate parameters. On Execute we verify the View is a Control and then invoke GoToState.

``` csharp
public abstract class ResultBase : IResult

{
    public event EventHandler<ResultCompletionEventArgs> Completed = delegate { };

    protected virtual void OnCompleted()
    {
        OnCompleted(new ResultCompletionEventArgs());
    }

    protected virtual void OnError(Exception error)
    {
        OnCompleted(new ResultCompletionEventArgs
        {
            Error = error
        });
    }

    protected virtual void OnCompleted(ResultCompletionEventArgs e)
    {
        Caliburn.Micro.Execute.OnUIThread(() => Completed(this, e));
    }

    public abstract void Execute(ActionExecutionContext context);
}
```

``` csharp
public class VisualStateResult : ResultBase
{
    private readonly string stateName;
    private readonly bool useTransitions;

    public VisualStateResult(string stateName, bool useTransitions = true)
    {
        this.stateName = stateName;

        this.useTransitions = useTransitions;
    }

    public string StateName
    {
        get { return stateName; }
    }

    public bool UseTransitions
    {
        get { return useTransitions; }
    }

    public override void Execute(ActionExecutionContext context)
    {
        if(!(context.View is Control))
            throw new InvalidOperationException("View must be a Control to use VisualStateResult");

        var view = (Control)context.View;

        VisualStateManager.GoToState(view, StateName, UseTransitions);

        OnCompleted();
    }
}
```

Once we have our custom IResult we can use it like:

``` csharp
yield return new VisualStateResult("Complete", useTransitions: false);
```




