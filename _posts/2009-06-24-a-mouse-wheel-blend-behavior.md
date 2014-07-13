---
layout: post
title: A Mouse Wheel Blend Behavior
tags: csharp silverlight
---

One fairly useful piece of functionality missing from Silverlight is the use of the Mouse Wheel, thankfully there's a lot of code out there about how to use the Html Bridge to receive DOM events and bring in Mouse Wheel functionality.

The trouble is that with these examples you need to wire up all the events and manually deal with the mouse wheel.

Silverlight / Blend Behaviors can help with this quite a bit by encapsulating all the required functionality in a simple drag and drop component.

As I've discussed before Blend Behaviors come in three flavors, Triggers, Actions and Behaviors. Triggers and Actions are great when you want to have both the action and trigger asseparate composable items but sometimes both the trigger and action are so intertwined that it doesn't make sense for them to be separated, this is where Behaviors come in, they really are pre-packaged pieces of functionality.

For this example we'll use the MouseWheelHelper class from the Deep Zoom Composer projects as this is one I've used a lot and fits nicely with what we'll be doing. The code for it is below.

``` csharp
public class MouseWheelHelper
{
    private static Worker MouseWheelWorker;
    private bool isMouseOver;
 
    public MouseWheelHelper(UIElement element)
    {
        if(MouseWheelWorker == null)
            MouseWheelWorker = new Worker();
 
        MouseWheelWorker.Moved += HandleMouseWheel;
 
        element.MouseEnter += HandleMouseEnter;
        element.MouseLeave += HandleMouseLeave;
        element.MouseMove += HandleMouseMove;
    }
 
    public event EventHandler<MouseWheelEventArgs> Moved;
 
    private void HandleMouseWheel(object sender, MouseWheelEventArgs args)
    {
        if(isMouseOver)
            Moved(this, args);
    }
 
    private void HandleMouseEnter(object sender, EventArgs e)
    {
        isMouseOver = true;
    }
 
    private void HandleMouseLeave(object sender, EventArgs e)
    {
        isMouseOver = false;
    }
 
    private void HandleMouseMove(object sender, EventArgs e)
    {
        isMouseOver = true;
    }
 
    private class Worker
    {
        public Worker()
        {
            if(!HtmlPage.IsEnabled)
                return;
 
            HtmlPage.Window.AttachEvent("DOMMouseScroll", HandleMouseWheel);
            HtmlPage.Window.AttachEvent("onmousewheel", HandleMouseWheel);
            HtmlPage.Document.AttachEvent("onmousewheel", HandleMouseWheel);
        }
 
        public event EventHandler<MouseWheelEventArgs> Moved;
 
        private void HandleMouseWheel(object sender, HtmlEventArgs args)
        {
            double delta = 0;
 
            var eventObj = args.EventObject;
 
            if(eventObj.GetProperty("wheelDelta") != null)
            {
                delta = ((double)eventObj.GetProperty("wheelDelta")) / 120;
 
 
                if(HtmlPage.Window.GetProperty("opera") != null)
                    delta = -delta;
            }
            else if(eventObj.GetProperty("detail") != null)
            {
                delta = -((double)eventObj.GetProperty("detail")) / 3;
 
                if(HtmlPage.BrowserInformation.UserAgent.IndexOf("Macintosh") != -1)
                    delta = delta * 3;
            }
 
            if(delta == 0 || Moved == null)
                return;
 
            var wheelArgs = new MouseWheelEventArgs(delta);
            Moved(this, wheelArgs);
 
            if(wheelArgs.Handled)
                args.PreventDefault();
        }
    }
}
```

The Behavior code is pretty simple, we're not adding any startling pieces of functionality here, the value we get is from setting this up as something that anyone using Blend can quickly add to their applications.

We'll be attaching our Behavior to any elements of ScrollViewer so we inherit from Behavior&lt;ScrollViewer&gt;. When the viewer is attached we create a MouseWheelHelper for the viewer and attach to the helper's Moved event. We also clean up the helper if the behavior is detached.

``` csharp
public class MouseWheelScrollBehavior : Behavior<ScrollViewer>
{
    private MouseWheelHelper helper;
 
    protected override void OnAttached()
    {
        base.OnAttached();
 
        helper = new MouseWheelHelper(AssociatedObject);
        helper.Moved += OnMouseWheelMoved;
    }
 
    private void OnMouseWheelMoved(object sender, MouseWheelEventArgs e)
    {
        var offset = AssociatedObject.VerticalOffset;
 
        AssociatedObject.ScrollToVerticalOffset(offset + (e.Delta * -10));
    }
 
    protected override void OnDetaching()
    {
        base.OnDetaching();
 
        helper.Moved -= OnMouseWheelMoved;
    }
}
```

On the mouse wheel moved event scroll the viewer by the scaled delta (you can change this constant in order to speed up or slow down the scroll speed).

From Blend you can now drag this Behavior from the Asset Library on to any ScrollViewer you want mouse wheel scrolling on. Sadly ListBox doesn't contain the appropriate scrolling methods to quickly add this in, however there should be nothing stopping you creating a new template forListBox that has this Behavior attached to the internal ScrollViewer.
