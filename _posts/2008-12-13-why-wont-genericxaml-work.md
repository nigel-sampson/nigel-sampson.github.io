---
layout: post
title: Why won't Generic.xaml work?
tags: csharp silverlight
---

The first time I did some control development for Silverlight I ran
into a major stumbling block, whatever I put into Generic.xaml just
wasn&#39;t showing up and none of the tutorials I could find were showing
anything different from what I was doing. After some digging through
Silverlight with Reflector and a few forum searches show up the culprit
... &quot;DefaultStyleKey&quot;.

I&#39;m not sure if this was introduced into
later versions of the Silverlight betas but it&#39;s certainly not
mentioned in a lot of tutorials on the web. OnApplyTemplate uses this
key to determine which style to look for in Generic.xaml. This allows
you to create controls that inherit from your custom control but still
use the same style.

For your control you&#39;ll want to override the DefaultStyleKey to the type of your object like follows.

``` csharp
public MyCustomControl()
{
    this.DefaultStyleKey = typeof(MyCustomControl);
}
```

It&#39;s tempting to do something like GetType(), but that&#39;ll be different for any subclasses so stick with a known type.


