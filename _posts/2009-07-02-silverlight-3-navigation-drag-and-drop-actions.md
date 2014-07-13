---
layout: post
title: Silverlight 3 Navigation Drag and Drop Actions
tags: csharp silverlight
---

So far I've covered building a Trigger "Double Click Trigger" and a Behavior "Mouse Wheel Behavior" so all that's left are Actions. Actions I believe are probably the most useful of the three sorts of Behaviors in Silverlight, the sheer amount of variation possible when composing these with events provides a lot of richfunctionality to the Blend user. The real power comes from having all this functionality at your drag and drop disposal, by quickly wiring this all together it leaves you more time for true functionality of your RIA.

For the Actions I want to create today I'm going to concentrate on the new navigation system in Silverlight 3. If you haven't taken a look at it I suggest you watch the video at [silverlight.net](http://silverlight.net/learn/learnvideo.aspx?video=187319). In the default examples we have to wire up the button clicks to change the current page ourselves. For a designer throwing together a prototype this can be awkward and slows everything down. Let's create a couple of Actionsthat'll speed this process up.

To begin with we need an Action that tells the Frame to navigate to a certain Uri. For this we need to associate the Action with two different elements, one, the element that will ultimately trigger the Action and two, the Frame that will do the navigation.

This is achieved by using the <em>TargetedTriggerAction&lt;T&gt;</em> abstract class. This allows us to define a Target of the Action where T is the constraint of the Target (and not the AssociatedObject). You can see how some of this works at "[Targeting Other Elements via the Blend 3 UI](http://blog.kirupa.com/?p=386)".

So for our Action to navigate on the Frame we inherit from **TargetedTriggerAction&lt;Frame&gt;**. The rest of the code is pretty simple, we expose a Uri property and when the action is invoked we call the Navigate method on the Target Frame.

``` csharp
public class NavigateFrameAction : TargetedTriggerAction<Frame>
{
    public Uri Uri
    {
        get;
        set;
    }
 
    protected override void Invoke(object parameter)
    {
        Target.Navigate(Uri);
    }
}
```

The other Action we're going to require is one that sits in our navigation Pages and tells the containing Frame to do the navigation. In theory we could build this using a **TargetedTriggerAction&lt;Page&gt;** but this relies on the Action being the same .xaml as the Page in order to target it. We could end up having multiple levels of controls so we'll need to walk to the visual tree ourselves to find the containing page. Thanks toVisualTreeHelper this is a pretty simple task. Our Navigation action inherits from  <em>TriggerAction&lt;UIElement&gt;</em> as we want to make this as applicable as possible. Again pretty simple code, on Invoke we locate the Page and use its NavigationService to trigger the Navigation.

``` csharp
public class NavigateAction : TriggerAction<DependencyObject>
{
    public Uri Uri
    {
        get;
        set;
    }
 
    protected override void Invoke(object parameter)
    {
        var page = FindContainingPage(AssociatedObject);
 
        if(page == null)
            throw new InvalidOperationException("Could not find the containing System.Windows.Controls.Page in the visual tree.");
 
        page.NavigationService.Navigate(Uri);
    }
 
    protected static Page FindContainingPage(DependencyObject associatedObject)
    {
        var current = associatedObject;
 
        while(!(current is Page))
        {
            current = VisualTreeHelper.GetParent(current);
 
            if(current == null)
                return null;
        }
 
        return (Page)current;
    }
}
```
![Frame action](/content/images/posts/frame-action.png)

Using them is very simple, drag either action onto the element you want triggering the page navigation. Type in the uri you want to navigate to and you're done.

![Visual Tree](/content/images/posts/visual-tree.png)
