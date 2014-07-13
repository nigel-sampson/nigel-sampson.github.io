---
layout: post
title: Extending the ListBox control
tags: csharp windows-phone
---

I'm currently going through a process of bringing together code and controls I've been using in the last few applications and compiling my own "toolkit". The idea will be a very opinionated way of building WP7 applications rather than a generic control library. Some parts will be controls like the "empty list box" discussed in this post and other parts will be extensions to the Caliburn.Micro MVVM framework for how I like to work.

The first control is an extension of the ListBox control to display certain content when there are no items in the list. It's a simple extension but most of the time we forget to add this functionality especially when we're dealing with lots of lists in our application.

The first thing we'll do is create our class ListBox (not the best name, but no other one like EmptyListBox suites, thankfully we have namespaces for conflicting names) obviously inheriting from the system ListBox, we'll had two dependency properties, EmptyContent and EmptyContentTemplate. This will allow two different ways of defining the content to display, much like Button or ContentControl.

``` csharp
public static readonly DependencyProperty EmptyContentTemplateProperty =
    DependencyProperty.Register("EmptyContentTemplate", typeof(DataTemplate), typeof(ListBox), null);
 
public static readonly DependencyProperty EmptyContentProperty =
    DependencyProperty.Register("EmptyContent", typeof(object), typeof(ListBox), null);
 
public ListBox()
{
    DefaultStyleKey = typeof(ListBox);
}
 
 
public object EmptyContent
{
    get
    {
        return GetValue(EmptyContentProperty);
    }
    set
    {
        SetValue(EmptyContentProperty, value);
    }
}
 
public DataTemplate EmptyContentTemplate
{
    get
    {
        return (DataTemplate)GetValue(EmptyContentTemplateProperty);
    }
    set
    {
        SetValue(EmptyContentTemplateProperty, value);
    }
}
```

Our default Style in Generic.xaml will be pretty simple.

``` xml
<Style TargetType="controls:ListBox">
    <Setter Property="Background" Value="Transparent"/>
    <Setter Property="Foreground" Value="{StaticResource PhoneForegroundBrush}"/>
    <Setter Property="ScrollViewer.HorizontalScrollBarVisibility" Value="Disabled"/>
    <Setter Property="ScrollViewer.VerticalScrollBarVisibility" Value="Auto"/>
    <Setter Property="BorderThickness" Value="0"/>
    <Setter Property="BorderBrush" Value="Transparent"/>
    <Setter Property="Padding" Value="0"/>
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="controls:ListBox">
                <Grid>
                    <ContentPresenter x:Name="EmptyContent" Content="{TemplateBinding EmptyContent}" ContentTemplate="{TemplateBinding EmptyContentTemplate}" RenderTransformOrigin="0.5,0.5" >
                        <ContentPresenter.RenderTransform>
                            <CompositeTransform/>
                        </ContentPresenter.RenderTransform>
                    </ContentPresenter>
                    <ScrollViewer x:Name="ScrollViewer" BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}" Background="{TemplateBinding Background}" Foreground="{TemplateBinding Foreground}" Padding="{TemplateBinding Padding}">
                        <ItemsPresenter/>
                    </ScrollViewer>
                </Grid>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

We'll then override the OnItemsChanged to add our new behaviour when items are added or removed.

There are a few different ways we can implement the required functionality each with their own pros and cons. 

### Using Control Template Parts

The first is using Template Parts, this allows controls to tell developers which sub-controls they expect in the control template. The can then interact with them using the FindTemplateChild method. The ListBox control already defines a ScrollViewer as a part, we would then add a ContentPresenter as a second part. In our overriden method we could then set the Visibility of each control appropriately. This approach is the simplest and the code is as follows.

``` csharp
private ContentControl emptyContent;
private ScrollViewer scrollViewer;
 
public override void OnApplyTemplate()
{
    base.OnApplyTemplate();
 
    emptyContent = (ContentControl)GetTemplateChild("EmptyContent");
    scrollViewer = (ScrollViewer)GetTemplateChild("ScrollViewer");
}
 
protected override void OnItemsChanged(NotifyCollectionChangedEventArgs e)
{
    base.OnItemsChanged(e);
 
    ApplyTemplate();
 
    var hasItems = Items.Count > 0;
 
    emptyContent.Visibility = hasItems ? Visibility.Collapsed : Visibility.Visible;
    scrollViewer.Visibility = hasItems ? Visibility.Visible : Visibility.Collapsed;
}
```

### Using Visual States

The second approach is to use the VisualStateManager to hide and show parts of our control template. I like this approach better as it gives better flexibility to developers extending the control. By defining two states "Empty" and "NonEmpty" we put it back in the developers hands about how each state should look, rather than arbitrarily defining the behavior ourselves.  This approach makes the control code itself simpler than the first but we have more xaml in the Control Template due to having to define the new states.

``` csharp
protected override void OnItemsChanged(NotifyCollectionChangedEventArgs e)
{
    base.OnItemsChanged(e);
 
    ApplyTemplate();
 
    VisualStateManager.GoToState(this, Items.Count > 0 ? "NonEmpty" : "Empty", true);
}
```

The xaml for the Visual States is pretty long so I won't post it here but it will be available in the download available soon.

Over all I prefer the second approach due it's freedom and the ability to add animations. The default template has a simple "continuum" animation bringing the content into view.

[Download the source](https://bitbucket.org/nigel.sampson/compiled-experience-toolkit).
