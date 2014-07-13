---
layout: post
title: Silverlight 3 Behaviors &#58; Double Click Trigger
tags: csharp silverlight
---

I've been come really enamoured with the possibilities of Silverlight / WPF Behaviors, broadly they can be divided into three types, Triggers, Actions and Behaviors. The ability for non-programmers to compose behaviour and interactivity in Blend is very interesting.

Triggers as the name suggests are objects the ultimately trigger other Actions, because they're separated in this way it means you can compose different combinations of Triggers and Actions for a richer user experience. For this post we'll look at filling a gap in Silverlight by creating a Double Click Trigger which can be used to invoke all sorts of actions, and all through Blend!.

To create a trigger we need to reference the assemble "Microsoft.Expression.Interactivty", if you've installed the Blend 3 Preview in to the default location then you can find this assembly in "C:\Program Files\Microsoft Expression\Blend 3 Preview\Libraries\Silverlight". Triggers inherit from a class named TriggerBase&lt;T&gt; where T is the type of element the trigger can be attached to. This means that if our trigger is only appropriate to say Buttons then we would inherit from TriggerBase&lt;Button&gt;.

For our trigger we need access to the "MouseLeftButtonDown" event and we want to be able to apply this to as many different elements as we can so we'll inherit from TriggerBase&lt;UIElement&gt;. The two important methods when creating all Silverlight 3 Behaviours is the OnAttached and OnDetaching methods, this is where we can start to interact with the element (AssociatedObject) we've been dropped on to. In our case we want to attach to the MouseLeftButtonDown event and to be a good citizen we'll clean up after ourselves on detaching

``` csharp
public class DoubleClickTrigger : TriggerBase<UIElement>
{
    private readonly DispatcherTimer timer;
 
    public DoubleClickTrigger()
    {
        timer = new DispatcherTimer
        {
            Interval = new TimeSpan(0, 0, 0, 0, 200)
        };
 
        timer.Tick += OnTimerTick;
    }
 
    protected override void OnAttached()
    {
        base.OnAttached();
 
        AssociatedObject.MouseLeftButtonDown += OnMouseButtonDown;
    }
 
    protected override void OnDetaching()
    {
        base.OnDetaching();
 
        AssociatedObject.MouseLeftButtonDown -= OnMouseButtonDown;
 
        if(timer.IsEnabled)
            timer.Stop();
    }
 
    private void OnMouseButtonDown(object sender, MouseButtonEventArgs e)
    {
        if(!timer.IsEnabled)
        {
            timer.Start();
            return;
        }
 
        timer.Stop();
 
        InvokeActions(null);
    }
 
    private void OnTimerTick(object sender, EventArgs e)
    {
        timer.Stop();
    }
}
```

The rest of the code is pretty simple, when a user clicks the associated object we start a timer that runs for 200ms, if we record another click while the timer is running then we call InvokeActions. This triggers any actions the user has attached to our trigger.

To use our new Trigger we drag on any Action from our assets panel on to our element, in my little example I'm using a test MessageBoxAction. Then under the properties of the action we change the trigger from the default EventTrigger to our new DoubleClickTrigger.

![Double click trigger](/content/images/posts/blend.png)

``` xml
<Rectangle Height="80" Width="130" Canvas.Left="63" Canvas.Top="102">
    <i:Interaction.Triggers>
        <cx:DoubleClickTrigger>
            <cx:MessageBoxAction Message="Test"/>
        </cx:DoubleClickTrigger>
    </i:Interaction.Triggers>
</Rectangle>
```
