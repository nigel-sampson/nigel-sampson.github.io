---
layout: post
title: Announcing Spinkit.Xaml
tags: windows-apps spinkit
---

A few months ago I saw a cool CSS project making the rounds called [SpinKit][spinkit], a collection of CSS spinner styles created by [Tobias Ahlin][tobias] of GitHub. Getting rather bored with the standard spinning circles I thought I’d try and recreate these in Xaml as Styles for the Windows 8 Progress Ring control. 

The result is [Spinkit.Xaml][github] which turned out to be reasonably accurate as you can see from the comparison video.

<iframe width="500" height="375" src="//www.youtube.com/embed/Lr5EulwTYz0" frameborder="0" allowfullscreen></iframe>

It’s all available on source on [GitHub][github] and I’m also distributing the resource dictionary through [Nuget][nuget] to make it easy to import them into your app and apply to your ProgressRing controls.

```xml
<ResourceDictionary.MergedDictionaries>
    <ResourceDictionary Source="Spinkit.Styles.xaml" />
</ResourceDictionary.MergedDictionaries>

<ProgressRing IsActive="True" Style="{StaticResource RotatingPlaneProgressRingStyle}" />
```

So how difficult is it to port something like this across from CSS to Xaml? In some parts easy and other parts not so much. The core animation ideas around key frames and easing work identically so there’s no real problems there. One minor difference is that SpinKit specifies its key frames as percentages (at 50% the scale should be X) whereas Xaml uses absolute times so a little math is involved there but nothing too taxing.

We also want to use the same easing in the animation as CSS, we can represent that using a SplineDoubleKeyFrame and the KeySpline “0.42,0 0.58,1”. 

```xml
<SplineDoubleKeyFrame KeyTime="0:0:1.2" Value="-180" KeySpline="0.42,0 0.58,1"/>
```

All this is relatively simple, what makes it complicated is tweaking it to use it as a control template. Which means we should respect the developer in terms of Foreground, Background, Width and Height etc. 

The last two are the most problematic for a couple of reasons, it means we can’t use absolute sizes for any of the elements in the template. Heavy use of Grids help us create elements that will scale with the control itself, I suspect you could get some really interesting effects using non-square sizes. 

Lack of absolute sizes means we can’t use absolute values in any of the animations either. Thankfully for most of them it isn’t a problem as we’re mostly animating Scale, Rotation or Opacity. 

Chasing Dots however causes a lot of problems, mainly because we want to animate the square from one side of the control to the other. In Xaml however there isn’t a way to do a relative translate animation, they must be absolute. Windows Phone introduced a control RelativeAnimatingContentControl in the [PerformanceProgressBar][ppb] that addresses this problem. However we can't introduce new code with just a resource dictionary.

This ultimately means Chasing Dots is hard coded for 80 x 80 rather than stretching like the other styles.

Import the resource dictionary from [Nuget][nuget], check out the source on [GitHub][github] and have fun!

[spinkit]:http://tobiasahlin.com/spinkit/
[tobias]: https://twitter.com/tobiasahlin
[github]: https://github.com/nigel-sampson/spinkit-xaml
[nuget]: https://www.nuget.org/packages/SpinKit.Xaml/
[ppb]: http://www.jeff.wilcox.name/2010/08/performanceprogressbar/
