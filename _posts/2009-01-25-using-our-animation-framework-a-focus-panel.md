---
layout: post
title: Using our animation framework, a focus panel
tags: csharp silverlight
---

Now that we have the basics of an animation framework in place from the following posts &quot;[Beginning a Silverlight animation framework](/blog/posts/beginning-a-silverlight-animation-framework)&quot; and &quot;[Silverlight Animation Part 2 : Size](/blog/posts/silverlight-animation-part-2-size/)&quot; we can start to build something useful with it.

Building
an sort of useful control is going to mean a long post so please bear
with me, if you&#39;d like to see the end result you can at &quot;[Example Focus Panel](/examples/controls)&quot;.

Our control inherits from Canvas since we use the
Canvas.Left and Canvas.Top attached properties for the position
animation. The key to our control are two methods LoadItems and
UpdateItems. 

LoadItems works out the Row and Column indices
for each of the FocusPanelItems and attaches to their Focused event.
For this example we&#39;re just going to try and fit them best into an
approximate square. Feel free to tinker with this for other layouts.

``` csharp
private void LoadItems()
{
    if(Children.Count == 0)
        return;
 
    Columns = (int)Math.Ceiling(Math.Sqrt(Children.Count));
    Rows = (int)Math.Ceiling((double)Children.Count / Columns);
 
    for(var childIndex = 0; childIndex < Children.Count; childIndex++)
    {
        var element = Children[childIndex] as FocusPanelItem;
 
        if(element == null)
            throw new InvalidOperationException("Children must be " + typeof(FocusPanelItem).Name);
 
        element.Focused += OnItemFocussed;
 
        element.RowIndex = childIndex / Columns;
        element.ColumnIndex = childIndex % Columns;
    }
}
```

UpdateItems
adjusts the position and size of each of the children depending on
which is focused if any. If an update was triggered by an item being
focused on then we&#39;ll animate this update, otherwise we&#39;ll do a simple
layout of the items.

``` csharp
private void UpdateItems(bool animate)
{
    if(Children.Count == 0)
        return;
 
    if(FocussedPanel == null)
    {
        var columnWidth = ActualWidth / Columns;
        var columnHeight = ActualHeight / Rows;
 
        foreach(FocusPanelItem element in Children)
        {
            var left = element.ColumnIndex * columnWidth;
            var top = element.RowIndex * columnHeight;
 
            var width = columnWidth - element.Margin.Left - element.Margin.Right;
            var height = columnHeight - element.Margin.Top - element.Margin.Bottom;
 
            UpdateItem(element, animate, left, top, width, height);
        }
    }
    else
    {
        var left = 0.0d;
 
        for(var i = 0; i < Children.Count; i++)
        {
            var element = Children[i] as FrameworkElement;
 
            if(element == null)
                throw new InvalidOperationException("Children must be " + typeof(FocusPanelItem).Name);
 
            if(element != FocussedPanel)
            {
                var top = ActualHeight - UnfocussedRowHeight;
 
                var width = (ActualWidth / (Children.Count - 1)) - element.Margin.Left - element.Margin.Right;
                var height = UnfocussedRowHeight - element.Margin.Top - element.Margin.Bottom;
 
                UpdateItem(element, animate, left, top, width, height);
 
                left += ActualWidth / (Children.Count - 1);
            }
            else
            {
                var width = ActualWidth - element.Margin.Left - element.Margin.Right;
                var height = ActualHeight - UnfocussedRowHeight - element.Margin.Top - element.Margin.Bottom;
 
                UpdateItem(element, animate, 0, 0, width, height);
            }
        }
    }
}
 
private static void UpdateItem(FrameworkElement element, bool animate, double left, double top, double width, double height)
{
    if(animate)
    {
        element.AnimatePosition(left, top);
        element.AnimateSize(width, height);
    }
    else
    {
        element.Width = width;
        element.Height = height;
 
        SetLeft(element, left);
        SetTop(element, top);
    }
}
```

FocusPanelItem is a simple styled control using the [Parts model](http://scorbs.com/2008/06/11/parts-states-model-with-visualstatemanager-part-1-of)
to have a focus button that ultimately fires the Focused event. In
later posts I&#39;ll talk about adding visual state to this control.

``` csharp
[TemplatePart(Name = FocusButtonElement, Type = typeof(ButtonBase))]
public class FocusPanelItem : ContentControl
{
    private const string FocusButtonElement = "FocusButton";
    private ButtonBase focusButton;
    public event EventHandler Focused;
 
    public FocusPanelItem()
    {
        DefaultStyleKey = typeof(FocusPanelItem);
    }
 
    public int ColumnIndex { get; set; }
    public int RowIndex { get; set; }
 
    protected ButtonBase FocusButton
    {
        get { return focusButton; }
        set
        {
            if(focusButton == value)
                return;
 
            if(focusButton != null)
                focusButton.Click -= OnFocusButtonClicked;
 
            focusButton = value;
 
            if(focusButton != null)
                focusButton.Click += OnFocusButtonClicked;
        }
    }
 
    protected virtual void OnFocused(EventArgs e)
    {
        if(Focused != null)
            Focused(this, e);
    }
 
    public override void OnApplyTemplate()
    {
        base.OnApplyTemplate();
 
        FocusButton = GetTemplateChild(FocusButtonElement) as ButtonBase;
    }
 
    private void OnFocusButtonClicked(object sender, RoutedEventArgs e)
    {
        OnFocused(EventArgs.Empty);
    }
}
```

We define the default look and feel for the FocusPanelItem in our Generic.xaml file.

``` xml
<ResourceDictionary
   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
   xmlns:cx="clr-namespace:CompiledExperience.Core.Controls"
   >
    <Style TargetType="cx:FocusPanelItem">
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="cx:FocusPanelItem">
                    <Border Margin="5,5,5,5" BorderThickness="5" Background="White" CornerRadius="10,10,10,10">
                        <Border.BorderBrush>
                            <LinearGradientBrush EndPoint="0.5,1" StartPoint="0.5,0">
                                <GradientStop Color="#FF5885A4"/>
                                <GradientStop Color="#FFFFFFFF" Offset="1"/>
                            </LinearGradientBrush>
                        </Border.BorderBrush>
                        <Grid Margin="10,10,10,10">
                            <Grid.RowDefinitions>
                                <RowDefinition Height="20" />
                                <RowDefinition Height="*" />
                            </Grid.RowDefinitions>
                            <Button FontFamily="Webdings" FontSize="8" Foreground="#252F37" HorizontalAlignment="Right" x:Name="FocusButton" Content="n" Grid.Row="0" Width="20" />
                            <ContentPresenter Margin="2,2,2,2" Grid.Row="1" />
                        </Grid>
                    </Border>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
</ResourceDictionary>
```

As an example of use here&#39;s the xaml I&#39;m using for the example that you can find at &quot;[Example Focus Panel](/examples/controls)&quot;.

You can also download the full code listing for this at &quot;[Focus Panel code](/content/downloads/compiledexperience.focuspanel.zip)&quot;. I didn&#39;t include all the code above but you can see it all here.
</p>

