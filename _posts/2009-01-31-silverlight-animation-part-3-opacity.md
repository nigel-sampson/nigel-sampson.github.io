---
layout: post
title: Silverlight Animation Part 3&#58; Opacity
tags: csharp silverlight
---

Time to extend our burgeoning framework with the ability to programmatically
animate an items opacity. It&#39;s pretty quick and I hope easy to follow
code, we&#39;re not using key frames or any splines to animate this, just a
simple linear animation. Here&#39;s the code.

``` csharp
public class OpacityAnimation : AnimationBase
{
    public static TimeSpan DefaultDuration = TimeSpan.FromMilliseconds(750);
 
    public OpacityAnimation(double opacity)
        : this(opacity, DefaultDuration)
    {
 
    }
 
    public OpacityAnimation(double opacity, TimeSpan duration)
    {
        Opactity = opacity;
        Duration = duration;
    }
 
    public double Opactity
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
 
        var opacityAnimation = new DoubleAnimation();
 
        Storyboard.SetTargetProperty(opacityAnimation, new PropertyPath("(FrameworkElement.Opacity)"));
 
        storyboard.Children.Add(opacityAnimation);
 
        return storyboard;
    }
 
    protected override void ApplyValues(Storyboard storyboard)
    {
        if(storyboard == null)
            throw new ArgumentNullException("storyboard");
 
        var opacityAnimation = storyboard.Children[0] as DoubleAnimation;
 
        opacityAnimation.To = Opactity;
        opacityAnimation.Duration = Duration;
    }
}
```

Now that we have that in place we can put together four very useful extension methods.

``` csharp
public static void FadeOut(this FrameworkElement element)
{
    element.Fade(0.0);
}
 
 
public static void FadeIn(this FrameworkElement element)
{
    element.Fade(1.0);
}
 
 
public static void Fade(this FrameworkElement element, double opacity)
{
    new OpacityAnimation(opacity).Apply(element);
}
 
public static void CrossFade(this FrameworkElement elementToFadeOut, FrameworkElement elementToFadeIn)
{
    FadeOut(elementToFadeOut);
    FadeIn(elementToFadeIn);
}
```

And here&#39;s the code to the example which you can find at &quot;[Silverlight Animation Examples](/examples/animation)&quot;.

``` csharp
if(leftFadedOut)
{
    Right.CrossFade(Left);
}
else
{
    Left.CrossFade(Right);
}
 
leftFadedOut = !leftFadedOut;
```

