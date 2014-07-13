---
layout: post
title: Beginning a Silverlight Animation framework
tags: csharp silverlight
---

Silverlight has an excellent animation system, built on top of storyboards and various sorts of animation classes. 

In
essence a storyboard is a collection of animations, these are all
subclassed from the System.Windows.Media.Animation.Timeline class.
Obivously the ColorAnimation classes are for animation a colour and so
on. For our basic examples we&#39;ll be working with DoubleAnimation&#39;s
which we&#39;ll use to manipulate an object&#39;s location and size.

So
whats the result we&#39;re after? Ideally it&#39;ll be a syntax that allows
animation of all objects not just ones that have a specific subclass,
something like this.

``` csharp
Target.AnimatePosition(Left.Value, Top.Value);
```

You can see the end result of this post at [Silverlight Animation Examples](/examples/animation)

We
also want to have a couple of behavioural constraints, that if an
animation (of the same sort) is running then it&#39;s halted and the new
one is started. We also want to reuse resources as effectively as
possible, not creating new storyboards and animations unless we have
to, this will speed the animation up and ensure we don&#39;t leak resources
and memory.

We&#39;ll need an AnimationBase class that&#39;ll handle the rules we
discussed above. This class will have abstract members for creating the
storyboard for this animation and because we want to reuse storyboards
between animations we&#39;ll seperate the values of animation from the
storyboard itself.

Overall our process for applying an animation to an element will be as follows:

1. Does the element have the storyboard for this animation in it&#39;s resouce collection.
2. If so they we pause that animation.
3. If not we create our new storyboard, add it the resource collection and set it&#39;s target.
4. We apply the values of the animation to the storyboard.
5. Begin the animation.

The code for Animation base is below:

``` csharp
public abstract class AnimationBase
{
    public event EventHandler AnimationCompleted;
 
    protected virtual void OnAnimationCompleted(EventArgs e)
    {
        if(AnimationCompleted != null)
            AnimationCompleted(this, e);
    }
 
    protected virtual string ResourceKey
    {
        get
        {
            return GetType().FullName;
        }
    }
 
    protected abstract Storyboard CreateStoryboard();
    protected abstract void ApplyValues(Storyboard storyboard);
 
    public virtual void Apply(FrameworkElement element)
    {
        if(element == null)
            throw new ArgumentNullException("element");
 
        Storyboard storyboard = null;
 
        if(element.Resources.Contains(ResourceKey))
        {
            storyboard = element.Resources[ResourceKey] as Storyboard;
            storyboard.Pause();
        }
        else
        {
            storyboard = CreateStoryboard();
            element.Resources.Add(ResourceKey, storyboard);
 
            foreach(var timeline in storyboard.Children)
            {
                Storyboard.SetTarget(timeline, element);
            }
        }
 
        ApplyValues(storyboard);
 
        storyboard.Completed += OnStoryboardCompleted;
        storyboard.Begin();
    }
 
    protected virtual void OnStoryboardCompleted(object sender, EventArgs e)
    {
        OnAnimationCompleted(EventArgs.Empty);
    }
}
```

Now
we have AnimationBase we&#39;ll build our first actual animation,
PositionAnimation. Our storyboard will have two animations, one for the
left property and the other for the top property, I&#39;ve made them both
using KeyFrames in order to give them a bit of spice rather than just
linear animations. The important thing to note is that we haven&#39;t set
any values on the animation about where we&#39;re moving the element to.
This will come in the ApplyValues overload, as mentioned before this is
because we want to reuse storyboards for multiple animations. We&#39;ll
also add some constructor arguments for configuring the position and
duration of the animation.

The code for PositionAnimation is below:

``` csharp
public class PositionAnimation : AnimationBase
{
    public static TimeSpan DefaultDuration = TimeSpan.FromMilliseconds(750);
 
    public PositionAnimation(double left, double top)
        : this(left, top, DefaultDuration)
    {
 
    }
 
    public PositionAnimation(double left, double top, TimeSpan duration)
    {
        Left = left;
        Top = top;
        Duration = duration;
    }
 
    public double Left
    {
        get;
        private set;
    }
 
    public double Top
    {
        get;
        private set;
    }
 
    public TimeSpan Duration
    {
        get;
        private set;
    }
 
    protected override void ApplyValues(Storyboard storyboard)
    {
        if(storyboard == null)
            throw new ArgumentNullException("storyboard");
 
        var leftAnimation = storyboard.Children[0] as DoubleAnimationUsingKeyFrames;
        var topAnimation = storyboard.Children[1] as DoubleAnimationUsingKeyFrames;
 
        leftAnimation.KeyFrames[0].Value = Left;
        leftAnimation.KeyFrames[0].KeyTime = KeyTime.FromTimeSpan(Duration);
        topAnimation.KeyFrames[0].Value = Top;
        topAnimation.KeyFrames[0].KeyTime = KeyTime.FromTimeSpan(Duration);
    }
 
    protected override Storyboard CreateStoryboard()
    {
        var storyboard = new Storyboard();
 
        var leftAnimation = new DoubleAnimationUsingKeyFrames();
        var topAnimation = new DoubleAnimationUsingKeyFrames();
 
        Storyboard.SetTargetProperty(leftAnimation, new PropertyPath("(Canvas.Left)"));
        Storyboard.SetTargetProperty(topAnimation, new PropertyPath("(Canvas.Top)"));
 
        storyboard.Children.Add(leftAnimation);
        storyboard.Children.Add(topAnimation);
 
        leftAnimation.KeyFrames.Add(new SplineDoubleKeyFrame()
        {
            KeySpline = new KeySpline()
            {
                ControlPoint1 = new Point(0.528, 0),
                ControlPoint2 = new Point(0.142, 0.847)
            }
        });
 
        topAnimation.KeyFrames.Add(new SplineDoubleKeyFrame()
        {
            KeySpline = new KeySpline()
            {
                ControlPoint1 = new Point(0.528, 0),
                ControlPoint2 = new Point(0.142, 0.847)
            }
        });
 
        return storyboard;
    }
}
```

With
our animation class in place we&#39;ll just create some extension methods
to use our animation classes and provide the syntax from the beginning.

``` csharp
public static void AnimatePosition(this FrameworkElement element, double left, double top)
{
    new PositionAnimation(left, top).Apply(element);
}
```

Over time I&#39;ll create more of these animation classes and post them and our finished result is a [Silverlight Animation Examples](/examples/animation)

