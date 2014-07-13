---
layout: post
title: Silverlight Animation Part 2&#58; Size
tags: csharp silverlight
---

Now we&#39;ll extend our animation system to be able to animate an elements size. You can see the end result of this post at [Silverlight Animation Examples](/examples/animation)

After the original post &quot;[Beginning a Silverlight Animation Framework](/blog/posts/beginning-a-silverlight-animation-framework.aspx)&quot;
I was asked a few times &quot;Why would I need something like this?&quot;. For a
lot of your one off animations it makes sense to use a storyboard
that&#39;s part of your user control resources, this way it&#39;s part of your
designers workflow in Blend.

But when you&#39;re creating controls that will be reused throughout the application (or even multiple applications) such as [Blacklight](http://www.codeplex.com/blacklight) then having quick access to programmatic animations becomes very handy. In later posts I&#39;ll show some applications of it.

So
size animations, these are going to be very similar to our position
animation, both are going to use two double key spline animations, the
only difference will be the property that will be the target of the
animation. In this case we&#39;ll be animating the &quot;Framework.Width&quot; &amp;
&quot;Framework.Height&quot; properties.

Below is the SizeAnimation class:

``` csharp
public class SizeAnimation : AnimationBase
{
    public static TimeSpan DefaultDuration = TimeSpan.FromMilliseconds(750);
 
    public SizeAnimation(double width, double height)
        : this(width, height, DefaultDuration)
    {
 
    }
 
    public SizeAnimation(double width, double height, TimeSpan duration)
    {
        Width = width;
        Height = height;
        Duration = duration;
    }
 
    public double Width
    {
        get;
        private set;
    }
 
    public double Height
    {
        get;
        private set;
    }
 
    public TimeSpan Duration
    {
        get;
        private set;
    }
 
    protected override Storyboard CreateStoryboard()
    {
        var storyboard = new Storyboard();
 
        var widthAnimation = new DoubleAnimationUsingKeyFrames();
        var heightAnimation = new DoubleAnimationUsingKeyFrames();
 
        Storyboard.SetTargetProperty(widthAnimation, new PropertyPath("(FrameworkElement.Width)"));
        Storyboard.SetTargetProperty(heightAnimation, new PropertyPath("(FrameworkElement.Height)"));
 
        storyboard.Children.Add(widthAnimation);
        storyboard.Children.Add(heightAnimation);
 
        widthAnimation.KeyFrames.Add(new SplineDoubleKeyFrame()
        {
            KeySpline = new KeySpline()
            {
                ControlPoint1 = new Point(0.528, 0),
                ControlPoint2 = new Point(0.142, 0.847)
            }
        });
 
        heightAnimation.KeyFrames.Add(new SplineDoubleKeyFrame()
        {
            KeySpline = new KeySpline()
            {
                ControlPoint1 = new Point(0.528, 0),
                ControlPoint2 = new Point(0.142, 0.847)
            }
        });
 
        return storyboard;
    }
 
    protected override void ApplyValues(Storyboard storyboard)
    {
        if(storyboard == null)
            throw new ArgumentNullException("storyboard");
 
        var widthAnimation = storyboard.Children[0] as DoubleAnimationUsingKeyFrames;
        var heightAnimation = storyboard.Children[1] as DoubleAnimationUsingKeyFrames;
 
        widthAnimation.KeyFrames[0].Value = Width;
        widthAnimation.KeyFrames[0].KeyTime = KeyTime.FromTimeSpan(Duration);
        heightAnimation.KeyFrames[0].Value = Height;
        heightAnimation.KeyFrames[0].KeyTime = KeyTime.FromTimeSpan(Duration);
    }
}
```

and here&#39;s our animation extension to use it.

``` csharp
public static void AnimateSize(this FrameworkElement element, double width, double height)
{
    new SizeAnimation(width, height).Apply(element);
}
```
You can see the end result of this post at [Silverlight Animation Examples](/examples/animation).


