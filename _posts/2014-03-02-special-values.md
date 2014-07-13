---
layout: post
title: Custom Special Values in Caliburn.Micro
tags: csharp caliburn.micro windows-apps windows-phone silverlight wpf
---

[Caliburn.Micro][cm] has features to allow you bind methods to actions with parameters. Check out the Action Parameters section of the [documentation][docs]. Part of this set of features includes "special values" such as $eventArgs or $dataContext, these allow an easy to access information contextual to the action. How can we use them?

I use these special values in a number of different ways. When binding a list of values the item template may contain a button with the following:

``` xml
<Button Style="{StaticResource ContentButton}" cm:Message.Attach="ViewSection($dataContext)">
```

This action bubbles up to the root model passing the item in the list the button was pressed for.

``` csharp
public void ViewSection(RepositorySection group) { }
```

Another is making use of the ItemClick functionality on GridView and ListView in Windows 8. This provides similar functionality without requiring buttons in your item templates. The difference is that clicked item is in the event args so I'd use syntax something similar to:

``` xml
<ListView cm:Message.Attach="[Event ItemClick] = [SelectItem($eventArgs)]" IsItemClickEnabled="True">
```

``` csharp
public void SelectItem(ItemClickEventArgs e) { }
```

Another example is around pinning tiles in Windows 8, for better user experience we want to pass an invocation point to SecondaryTile.RequestCreateAsync to ensure the popup is near the element that invoked it. You can get access to element using $source or the $executionContext.

``` csharp
public async void ToggleTile(ActionExecutionContext context)
{
    var transform = context.Source.TransformToVisual(null);
 
    var invocationPoint = transform.TransformPoint(new Point(0, 0));
    
    ...
}
```

While all these solutions solve the problem at hand they do make the code messier than I'd ideally like. The ItemClick example forces you view model to depend on a xaml event args object with an internal constructor. While the invocation point sample forces the view model to start dealing with UI elements and their layout on the screen.

So how do we solve these problems? We can actually create custom special values, this removes the weird dependencies on the view model and increases code reuse as the special values are only dealt with once. For the above examples we'd have the following code in our bootstrapper (or application) depending platform.

``` csharp
MessageBinder.SpecialValues.Add("$invocationpoint", c => c.Source.TransformToVisual(null).TransformPoint(new Point()));

MessageBinder.SpecialValues.Add("$clickeditem", c => ((ItemClickEventArgs)c.EventArgs).ClickedItem);
```

Important points to note is that the key's must start with $ and be lowercase (they don't need to be in the xaml though). 
We can now change our xaml for our examples to the following.

``` xml
<ListView cm:Message.Attach="[Event ItemClick] = [SelectIssue($clickedItem)]" />
```

``` xml
<AppBarButton cm:Message.Attach="TogglePin($invocationPoint)" />
```

There's a myriad of ways this trick could possibly be put to use. I'm going back through old projects and looking where I'm depending on special values and seeing if they can be simplified. 

[docs]: https://caliburnmicro.codeplex.com/wikipage?title=All%20About%20Actions&referringTitle=Documentation
[cm]: https://github.com/BlueSpire/Caliburn.Micro
