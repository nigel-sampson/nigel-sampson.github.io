---
layout: post
title: Clamping Values
tags: csharp
---

In last week’s post on building the [iOS 7 parallax background in Windows Phone][ios] I commented on the fact I was surprised there wasn't a Clamp function on the Math class.

For those who aren't sure of the term it originates (I believe) out of computer graphics and ensures a value lies between a range of values. In essence it combines the Min and Max functions so that a simple implementation of Clamp could look like.

``` csharp
public static double Clamp(double value, double minimum, double maximum)
{
    return Math.Max(minimum, Math.Min(maximum, value));
}
```

Rather than write this method for each different type we can make use of the built in interface IComparable and create a generic clamp method that looks like:

``` csharp
public static T Clamp<T>(this T value, T minimum, T maximum)
    where T : IComparable<T>
{
    if (value.CompareTo(minimum) < 0)
        return minimum;
 
    if (value.CompareTo(maximum) > 0)
        return maximum;
 
    return value;
}
```

The great thing about it being generic off the IComparable interface is it works on any class implementing the interface including types like TimeSpan.

``` csharp
[TestMethod]
public void ClampTimeSpanWithValueBelowRangeReturnsMinimum()
{
    var value = TimeSpan.FromSeconds(10);
 
    var clamped = value.Clamp(TimeSpan.FromSeconds(20), TimeSpan.FromSeconds(30));
 
    Assert.AreEqual(TimeSpan.FromSeconds(20), clamped);
}
```

[ios]: http://compiledexperience.com/blog/posts/windows-phone-parallax-background

