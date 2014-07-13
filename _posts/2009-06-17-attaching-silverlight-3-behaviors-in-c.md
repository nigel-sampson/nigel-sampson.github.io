---
layout: post
title: Attaching Silverlight 3 Behaviors in C#
tags: csharp silverlight
---

For the most part we'll be adding or manipulating Silverlight / WPF Behaviors from Blend, but occasionally we'll need to manage these from code. Overall it's a pretty simple process. The main class we need to deal with is *Microsoft.Expression.Interactivity.Interaction*, from here we can retrieve the Behaviors and Triggers collections for a DependencyObject.

To attach a standard Behavior we use the following code.

``` csharp
var scrollBehavior = new MouseWheelScrollBehavior();
 
var behaviours = Interaction.GetBehaviors(Viewer);   
 
behaviours.Add(scrollBehavior);
```

To remove an attached behavior we use the following.

``` csharp
behaviours.Remove(scrollBehavior);
```

Managing triggers and actions are a two step process. In Blend elements have triggers and triggers have actions. You can have multiple actions per trigger, and multiple triggers per element. First we create our trigger and our action, we then add our action to the trigger's actions collection and finally the trigger to the element's triggers collection.

``` csharp
var trigger = new RandonTimerTrigger();
var action = new MessageBoxAction();
 
trigger.Actions.Add(action);
 
var triggers = Interaction.GetTriggers(Viewer);
 
triggers.Add(trigger);
```

Removing an action from a trigger or a trigger from an element is similar process to behaviors.

``` csharp
triggers.Remove(trigger);
```
