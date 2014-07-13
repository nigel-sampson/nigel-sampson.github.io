---
layout: post
title: Updated Silverlight Mouse Wheel Behavior
tags: csharp silverlight
---

A while ago I showed how you could hook into the DOM's MouseWheel events and create a Behavior to attach Mouse Wheel scrolling to any ScrollViewer. You can read about it from the sidebar on the right.

Since then Silverlight 3 has been released and along with it we have Mouse Wheel support backed in, we can now build a much simpler Behavior to add Mouse Wheel functionality toScrollViewer. Here it is.

``` csharp
public class MouseWheelScrollBehavior : Behavior<ScrollViewer>
{
    protected override void OnAttached()
    {
        base.OnAttached();
 
        AssociatedObject.MouseWheel += OnMouseWheel;
    }
 
    protected override void OnDetaching()
    {
        base.OnDetaching();
 
        AssociatedObject.MouseWheel -= OnMouseWheel;
    }
 
    private void OnMouseWheel(object sender, MouseWheelEventArgs e)
    {
        var offset = AssociatedObject.VerticalOffset;
 
        AssociatedObject.ScrollToVerticalOffset(offset + (e.Delta * -0.5));
    }
}
```
