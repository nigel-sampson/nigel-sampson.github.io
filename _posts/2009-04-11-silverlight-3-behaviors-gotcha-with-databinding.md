---
layout: post
title: Silverlight 3 Behaviors &#58; Gotcha with Databinding
tags: silverlight
---

Sorry for the lack of posts but I&#39;m currently moving house and trying
to learn Silverlight 3. There&#39;s a lot of very interesting features
coming with Silverlight 3. One of the first things I tried playing with
was the behaviors built into Blend 3. The first behavior I looked to
build was an ExecuteCommandAction, the idea being I could bind the
ICommand of this behavior to my ViewModel. This way I could make use of
the various triggers available to execute the command. 

Then problems strike, The TriggerAction&lt;T&gt; class that our
ExecuteCommandAction will inherit from doesn&#39;t inherit from
FrameworkElement, we can&#39;t actually do any Databinding with this
element (actually that&#39;s not true, you can set up your binding
elsewhere on a different FrameworkElement and use the new ElementName
binding expression) but that doesn&#39;t help us. 

I have seen a few implmentations of this currently out there, these all
involve using reflection to find the property on the attached object&#39;s
DataContext. While it works it&#39;s not strongly typed and has to resort
to the reflection API (which I think you should avoid if possible).

This is something I&#39;d love to see changed during the Silverlight 3 Beta
as having the ability to do full data binding on these new behaviors
through the Blend 3 UI will make for some excellent functionality.

I hope to have more up soon on Silverlight 3.

