---
layout: post
title: Silverlight 4 and Behaviors
tags: csharp silverlight
---

I was pleasantly surprised to see in the [Silverlight 4 feature list](http://timheuer.com/blog/archive/2009/11/18/whats-new-in-silverlight-4-complete-guide-new-features.aspx) that it will be possibly to setup Bindings on DependencyObjects (as opposed to FrameworkElements) in the same way you can with WPF. This greatly affects in my opinion the benefits of Blend Triggers, Actions and Behaviors, being able to bind them to other elements or a ViewModel (if using the MVVM pattern) lets you set up some really cool stuff.

A while ago I talked about [using Blend Behaviors to wire together triggers to commands](/blog/posts/blendable-mvvm-commands-and-nehaviors). It was possible, but required a lot of wiring code in order to expose Binding objects and listen for changes. With Silverlight 4 it becomes a lot simpler and lot more readable. The full code is for ExecuteTriggerAction is:

``` csharp
public class ExecuteCommandAction : TriggerAction<DependencyObject>
{
    public static readonly DependencyProperty CommandProperty =
        DependencyProperty.Register("Command", typeof(ICommand), typeof(ExecuteCommandAction), null);
 
    public static readonly DependencyProperty CommandParameterProperty =
        DependencyProperty.Register("CommandParameter", typeof(object), typeof(ExecuteCommandAction), null);
 
    public ICommand Command
    {
        get
        {
            return (ICommand)GetValue(CommandProperty);
        }
        set
        {
            SetValue(CommandProperty, value);
        }
    }
 
    public object CommandParameter
    {
        get
        {
            return GetValue(CommandParameterProperty);
        }
        set
        {
            SetValue(CommandParameterProperty, value);
        }
    }
 
    protected override void Invoke(object parameter)
    {
        if(Command != null && Command.CanExecute(CommandParameter))
            Command.CanExecute(CommandParameter);
    }
}
```
