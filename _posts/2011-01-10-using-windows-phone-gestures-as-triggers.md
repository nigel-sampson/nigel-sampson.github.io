---
layout: post
title: Using Windows Phone Gestures as Triggers
tags: csharp windows-phone
---

The [Silverlight Toolkit for Windows Phone 7](http://silverlight.codeplex.com/) made it really useful for apps to deal with simple gestures such as tap, pinch and swipe. Wiring these gestures to event handlers is incredibly simple with the code below. 

``` xml
<toolkit:GestureService.GestureListener>
    <toolkit:GestureListener DoubleTap="OnDoubleTap" />
</toolkit:GestureService.GestureListener>
```

Sometimes however we'll want to use gestures to trigger actions in Blend, this could be for simple interactivity *when tapped trigger a storyboard* to wiring up an Model, View, ViewModel framework such as Caliburn Micro or MVVMLight. To do this we'll need to create some Triggers.

For gestures that don't provide extra information such as Tap, Hold and Double Tap a simple trigger will work for all three, we'll provide and enumeration to let the user select what gesture they require.

``` csharp
public enum Gesture
{
    Tap,
    Hold,
    DoubleTap
}

public class GestureServiceTrigger : TriggerBase<FrameworkElement>
{
    public Gesture Gesture
    {
        get; set;
    }
 
    protected override void OnAttached()
    {
        var listener = GestureService.GetGestureListener(AssociatedObject);
 
        switch(Gesture)
        {
            case Gesture.Tap:
                listener.Tap += OnGesture;
                break;
            case Gesture.Hold:
                listener.Hold += OnGesture;
                break;
            case Gesture.DoubleTap:
                listener.Flick += OnGesture;
                break;
        }
    }
 
    protected override void OnDetaching()
    {
        var listener = GestureService.GetGestureListener(AssociatedObject);
 
        switch(Gesture)
        {
            case Gesture.Tap:
                listener.Tap -= OnGesture;
                break;
            case Gesture.Hold:
                listener.Hold -= OnGesture;
                break;
            case Gesture.DoubleTap:
                listener.Flick -= OnGesture;
                break;
        }
    }
 
    private void OnGesture(object sender, GestureEventArgs e)
    {
        e.Handled = true;
 
        InvokeActions(e);
    }
}
```

<span class="alignleft frame"><img alt="Windows Phone 7 Gesture as a Trigger" src="/content/images/posts/gesture.jpg"/></span>

Once it's compiled you can use Blend to drag on any Action from the Assets Panel, then select the GestureTrigger for the Action. 

For the Flick gesture we'll create a separate TriggerAction that also lets the user select the Using Windows Phones Gestures as Triggersdirection of the flick.

``` csharp
public class FlickGestureServiceTrigger : TriggerBase<FrameworkElement>
{
    public Orientation Direction
    {
        get; set;
    }
 
    protected override void OnAttached()
    {
        var listener = GestureService.GetGestureListener(AssociatedObject);
 
        listener.Flick += OnFlick;
    }
 
    protected override void OnDetaching()
    {
        var listener = GestureService.GetGestureListener(AssociatedObject);
 
        listener.Flick -= OnFlick;
    }
 
    private void OnFlick(object sender, FlickGestureEventArgs e)
    {
        if(e.Direction != Direction)
            return;
 
        e.Handled = true;
 
        InvokeActions(e);
    }
}
```

