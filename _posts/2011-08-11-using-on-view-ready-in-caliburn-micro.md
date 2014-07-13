---
layout: post
title: Using on View Ready in Caliburn Micro
tags: csharp windows-phone caliburn.micro
---

One of the [recommendations from Microsoft](http://msdn.microsoft.com/en-us/library/ff967560.aspx#BKMK_Startup) for better performance from your WP7 application is to begin work on your page in the first LayoutUpdated event after OnNavigatedTo.

In MVVM frameworks this wasn't the easiest to achieve, [Ben Gracewood](http://www.ben.geek.nz/2011/03/wp7-background-loading-with-caliburn-and-coroutines/) wrote an awesome post on how you can achieve this in [Caliburn Micro](http://caliburnmicro.codeplex.com/) 1.0 by creating your own FrameAdapter. I recommend reading this post for this as well as its discussion on co-routines (one of the lesser used features of Caliburn Micro).

In the 1.1 release the process becomes a little easier, the class ViewAware (from which Screen derives) has a method called OnViewReady that will be called on the first layout updated event after OnNavigatedTo.

``` csharp
protected internal virtual void OnViewReady(object view);
```

There's one small problem this this method, it's no a co-routine, something I'm making more and more use of in my projects. So how can we go about extending this?

Thankfully it's pretty easy to invoke Actions in CM, so on the OnViewReady method we invoke an action of the same name passing in the view (this is important since the results in the co-routine may want to interact with the view).

We can then create a virtual OnViewReady co-routine method that does nothing. All done!

``` csharp
public abstract class ViewModelBase : Screen

{
    protected override void OnViewReady(object view)
    {
        Caliburn.Micro.Action.Invoke(this, "OnViewReady", (DependencyObject)view);
    }

    public virtual IEnumerable<IResult> OnViewReady()
    {
        yield break;
    }
}
```

On a personal note, I'll be out of the country for the next five weeks getting married to a fantastic canadian girl so the blog will obviously be a little quiet. I may need to sneak back for a post or two since a semi secret Windows Phone project I've been working on is going to be launched by the client in the next month.
