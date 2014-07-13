---
layout: post
title: Silverlight Animation Part 4&#58; Transforms
tags: csharp silverlight
---

Silverlight has a pretty cool transform model that you can apply simple
transforms (and if you&#39;re good at matrix math some complicated ones) to
elements. You can apply atransform (or a group of transforms) by assigning to the elements RenderTransform property. We&#39;ll build a generic transform animation and then build on top of it for a RotationAnimation. As always there are examples on the [Silverlight Animation Examples](/examples/animation). 

Animating
an elements transform is slight different to other animations we&#39;ve
done so far. Before we were simply animating properties that always
existed on the element we were seeking to animate. For transformations
we&#39;ll need to animate a property on an element that may or may not
exist and it would be good to preserve any existing transformations on
the element.

What this means is that the Storyboard we create for each element will be different and we&#39;ll also need to do some preparation work on the element. To deal with this I&#39;ve altered the signature of CreateStoryboard to take the element we&#39;re creating the storyboard for. This won&#39;t affect any of the current animations such as Size and Opactity other than to update the method signature. The storyboards they create will not differ per element

``` csharp
public abstract class TransformAnimation<T> : AnimationBase where T : Transform, new()
{
    protected TransformAnimation(TimeSpan duration)
        : base(duration)
    {
 
    }
 
    protected TransformAnimation()
        : this(DefaultDuration)
    {
 
    }
 
    protected PropertyPath PrepareElement(FrameworkElement element)
    {
        if(element == null)
            throw new ArgumentNullException("element");
 
        if(element.RenderTransform == null)
        {
            element.RenderTransform = new TransformGroup();
        }
 
        if(element.RenderTransform is TransformGroup)
        {
            var group = (TransformGroup)element.RenderTransform;
            group.Children.Add(new T());
 
            return new PropertyPath(String.Format("(UIElement.RenderTransform).(TransformGroup.Children)[{0}]", group.Children.Count - 1));
        }
 
        if(!(element.RenderTransform is TransformGroup))
        {
            var transform = element.RenderTransform;
            var group = new TransformGroup();
 
            group.Children.Add(transform);
            group.Children.Add(new T());
 
            element.RenderTransform = group;
 
            return new PropertyPath(String.Format("(UIElement.RenderTransform).(TransformGroup.Children)[{0}]", group.Children.Count - 1));
        }
 
 
        throw new InvalidOperationException("Could not get prepare element for " + typeof(T).Name);
    }
}
```
The preparation
we need to do is to ensure that an element of the appropriate transform
is available to animate, in order to preserve any existing
transformations are preserved we&#39;ll always create new transform object.
If the element has no RenderTransform set we&#39;ll create a TransformGroup with a child Transform of our choice inside. 

If the RenderTransform is an existing Transform group we simply add our Transform to it. Once we&#39;ve determined the transform we need to add we return the PropertyPath
to it. This allows us to animate the correct element. Otherwise we&#39;ll
be following all the same pattern as any other of our animation
classes. We&#39;ll start with a RotationAnimation.

``` csharp
public RotationAnimation(double degrees)
        : this(degrees, DefaultDuration)
    {
 
    }
 
    public RotationAnimation(double degrees, TimeSpan duration)
        : base(duration)
    {
        Degrees = degrees;
    }
 
    public double Degrees
    {
        get;
        private set;
    }
 
    protected override Storyboard CreateStoryboard(FrameworkElement element)
    {
        var propertyPath = PrepareElement(element).Combine(new PropertyPath("(RotateTransform.Angle)"));
 
        var storyboard = new Storyboard();
 
        var animation = new DoubleAnimation();
 
        Storyboard.SetTargetProperty(animation, propertyPath);
 
        storyboard.Children.Add(animation);
 
        return storyboard;
    }
 
    protected override void ApplyValues(Storyboard storyboard)
    {
        if(storyboard == null)
            throw new ArgumentNullException("storyboard");
 
        var animation = (DoubleAnimation)storyboard.Children[0];
 
        animation.To = Degrees;
        animation.Duration = Duration;
    }
}
```

