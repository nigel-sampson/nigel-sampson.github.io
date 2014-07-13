---
layout: post
title: A Solution to Binding on Silverlight Behaviors
tags: csharp silverlight
---

I posted a while ago about when first playing with Silverlight 3
Behaviors that you can&#39;t data bind to them. The reason being is that
the base classes for Behaviors, Triggers and Actions are
DependencyObject which Silverlight (but not WPF apparently) doesn&#39;t
support data binding. 

While talking with some of the
Silverlight and Blend teams I&#39;ve found while this is a known issue it
won&#39;t be addressed in the Silverlight 3 timeframe. From what I gather
rather than changing Behaviours to inherit from FrameworkElement we may
see the data binding mechanisms changed to match with WPF.

There is an excellent post by Pete Blois &quot;[DataTrigger, Bindings on non-FrameworkElements, TypeConverters, DataStateBehavior &amp; DataStateSwitchBehavior](http://blois.us/blog/2009/04/datatrigger-bindings-on-non.html)&quot;
which shows how you can get bindings on non FrameworkElements in
Silverlight. To quickly sum up the solution involves exposing the
properties as Binding objects rather than type you&#39;d like to expose
(and a lot of trickery involving attached dependency properties).

While
this solution works (I&#39;ve been using it to create an
ExecuteCommandAction that I can bind Commands from my ViewModel) there
is a major problem with this approach.

Simply put you have to
make a decision when creating your Behavior do you want it to be
bindable or to use values from the designer. Which means you need to
create duplicate behaviours to get both sorts of functionality.

