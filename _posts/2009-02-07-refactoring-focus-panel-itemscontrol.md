---
layout: post
title: Refactoring Focus Panel&#58; ItemsControl
tags: csharp silverlight
---

So while the FocusPanel we built in &quot;[Using our animation framework, a focus panel](/blog/posts/ysing-our-animation-framework-a-focus-panel) &quot; it had a lot of room for improvement. The biggest to change are, the bad syntax required to add items to the panel in xaml
and the inability to bind data to the collection. Previously, anything
we wanted to add to the panel had to be declared within a FocusPanelItem which was the control that was animated and had the focus button&quot;. I forgot to include this in the previous post but the xaml for the example you can see at &quot; [Focus Panel example](/examples/controls)&quot; is as follows.

``` xml
<cx:FocusPanel x:Name="Items">
    <cx:FocusPanelItem Content="Example Text"/>
    <cx:FocusPanelItem>
        <cx:FocusPanelItem.Content>
            <Ellipse Fill="#FF5885A4" />
        </cx:FocusPanelItem.Content>
    </cx:FocusPanelItem>
    <cx:FocusPanelItem>
        <cx:FocusPanelItem.Content>
            <TextBox Text="Examplte TextBox" />
        </cx:FocusPanelItem.Content>
    </cx:FocusPanelItem>
    <cx:FocusPanelItem>
        <cx:FocusPanelItem.Content>
            <Button Content="Example Button"/>
        </cx:FocusPanelItem.Content>
    </cx:FocusPanelItem>
</cx:FocusPanel>
```

Pretty
verbose huh? So how can we change this?. Well a lot of the
functionality we&#39;re going to be needing comes hand in hand with the [ItemsControl](http://msdn.microsoft.com/en-us/library/system.windows.controls.itemscontrol%28VS.95%29.aspx). The ItemsControl is very very useful all by itself, used directly you define two templates. The ItemPanelTemplate defines the layout to display the items. This could be a StackPanel, Grid, Canvas or any other custom panel control you have. The ItemTemplate allows you to define a DataTemplate
to display the item you&#39;re binding. Between these two template we can
define a lot of what we want, but we need to go a step further.

Rather than inheriting FocusPanel from Canvas we&#39;ll need to inherit from ItemsControl.
But we still need a Canvas to do our layout (as we animate the
Canvas.Left and Canvas.Top attached properties for our position
animation). So the first thing we&#39;ll do is [define the DefaultStyleKey](/blog/posts/why-wont-genericxaml-work) in the constructor and set up a default ItemPanelTemplate in Generic.xaml.

``` csharp
public FocusPanel()
{
    DefaultStyleKey = typeof(FocusPanel);
    SizeChanged += OnSizeChanged;
 
    FocusPanelItems = new List<FocusPanelItem>();
}
```

``` xml
<Style TargetType="cx:FocusPanel">
    <Setter Property="ItemsPanel">
        <Setter.Value>
            <ItemsPanelTemplate>
                <Canvas/>
            </ItemsPanelTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

ItemsControl also has the concept of a container. Each item bound is
inserted into a container if necessary. If the item isn&#39;t a UIElement
then it&#39;ll create a very simple container (a ContentPresenter) and use
that to display the non-UIElement object. What we need to do is change
a little of this behaviour.

ItemsControl doesn&#39;t internally maintain a list of items so we&#39;ll need
to manually maintain this one. We also won&#39;t need to do anything on the
Loaded event so we can remove the handler for that. We can also remove the code we attach to the OnFocused event in LoadItems as we&#39;ll be doing that elsewhere.

``` csharp
private int Columns { get; set; }
private int Rows { get; set; }
private IList<FocusPanelItem> FocusPanelItems { get; set; }
private FocusPanelItem FocussedPanel { get; set; }
 
 
public FocusPanel()
{
    DefaultStyleKey = typeof(FocusPanel);
    SizeChanged += OnSizeChanged;
 
    FocusPanelItems = new List<FocusPanelItem>();
}
```

Now we need to tell the ItemsControl to create a FocusPanelItem for every item (unless the item is already a FocusPanelItem). We override IsItemItsOwnContainerOverride (I&#39;m not sure why they chose to append Override to the name but ok).

``` csharp
private int Columns { get; set; }
private int Rows { get; set; }
private IList<FocusPanelItem> FocusPanelItems { get; set; }
private FocusPanelItem FocussedPanel { get; set; }
 
 
public FocusPanel()
{
    DefaultStyleKey = typeof(FocusPanel);
    SizeChanged += OnSizeChanged;
 
    FocusPanelItems = new List<FocusPanelItem>();
}
```

We then override GetContainerForItemOverride which will create a new FocusPanelItem for the the currently bound item.

``` csharp
protected override DependencyObject GetContainerForItemOverride()
{
    return new FocusPanelItem();
}
```

Overriding PrepareContainerForItemOverride we want to keep the base behavior but also add our item to our internal collection, attach the OnFocussed event and then update our panels.

``` csharp
protected override void PrepareContainerForItemOverride(DependencyObject element, object item)
{
    base.PrepareContainerForItemOverride(element, item);
 
    var panelItem = (FocusPanelItem)element;
 
    panelItem.Focused += OnItemFocused;
 
    FocusPanelItems.Add(panelItem);
 
    LoadItems();
    UpdateItems(false);
}
```

And when an is removed we override ClearContainerForItemOverride which will remove our item from the collection, detach from the OnFocussed event and reupdate our panels.

``` csharp
protected override void ClearContainerForItemOverride(DependencyObject element, object item)
{
    var panel = (FocusPanelItem)element;
    panel.Focused -= OnItemFocused;
 
    FocusPanelItems.Remove(panel);
 
    if(FocussedPanel == panel)
        FocussedPanel = null;
 
    LoadItems();
    UpdateItems(false);
}
```

Overall our [example](/examples/controls)  hasn&#39;t changed much but we can now either bind to the panels ItemSource (though code or in xaml) or use xaml to add children to the panel. 

``` csharp
private void OnLoaded(object sender, RoutedEventArgs e)
{
    Items.ItemsSource = new string[] { "Item One", "Item Two", "Item Three" };
}
```

``` xml
<cx:FocusPanel x:Name="Items">
    <Ellipse Fill="#FF5885A4" />
    <TextBox Text="Examplte TextBox" />
    <Button Content="Example Button"/>
</cx:FocusPanel>
```

You can download the code for this at &quot;[FocusPanel example code](/content/downloads/CompiledExperience.FocusPanel.ItemsControl.zip)&quot;

