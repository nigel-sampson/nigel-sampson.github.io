---
layout: post
title: Styling the Focus Panel &#58; Visual State Manager
tags: csharp silverlight
---

The Focus Panel is starting come together but it&#39;ll be good to add some
more customizability to the controls. In this post we&#39;ll add the
ability to not only style the panel items but to give the different
states of the items their own style.

As always the finished product is viewed at &quot;[Focus Panel examples](/examples/controls)&quot;.

By default the ItemsControl doesn&#39;t expose a style for the containers it generates so what we create is a new Dependency Property named ItemStyle and apply it to the container in PrepareContainerForItemOverride.

``` csharp
public static readonly DependencyProperty ItemStyleProperty =
    DependencyProperty.Register("ItemStyle", typeof(Style), typeof(FocusPanel), null);
 
public Style ItemStyle
{
    get
    {
        return (Style)GetValue(ItemStyleProperty);
    }
    set
    {
        SetValue(ItemStyleProperty, value);
    }
}

if(ItemStyle != null)
    panelItem.Style = ItemStyle;
```

For a different appearance
per state we turn to the Visual State Manager built into Silverlight.
The first thing we&#39;ll do is set up the states that we&#39;re going to want.
States are divided up into State Groups and an object can be in a state
for each state group it has. A good example of this is the Check Box.
It&#39;ll have a checked and unchecked state as well as a mouse over and
non mouse over state. Logically the check box can be checked and be a
mouse over so the check states and the mouse states are divided intoseparate groups. 

For us however one simple state group with three states is fine. Sadly the VisualStateManager uses strings to represent states which I&#39;m not a giant fan of but such as life. We use attributes to tell VisualStateManager (and Blend) what states are available.

``` csharp
[TemplatePart(Name = FocusButtonElement, Type = typeof(ButtonBase))]
[TemplateVisualState(Name = NormalState, GroupName = CommonStateGroup)]
[TemplateVisualState(Name = UnfocusedState, GroupName = CommonStateGroup)]
[TemplateVisualState(Name = FocusedState, GroupName = CommonStateGroup)]
public class FocusPanelItem : ContentControl
{
    private const string FocusButtonElement = "FocusButton";
    private const string NormalState = "Normal";
    private const string UnfocusedState = "Unfocused";
    private const string FocusedState = "Focused";
    private const string CommonStateGroup = "CommonStates";
```

Now we&#39;re going to do is create a simple enum for us to track the state of each item, add a property to the FocusPanelItem and modify OnItemFocused
to update the state of each of the items whenever there&#39;s a state
change. The property is important as when an item state changes it
needs to inform theVisualStateManager. It does this with the GoToState method, we&#39;ll the value of the enum ToString to get the state name (part of why I don&#39;t like magic strings) and tell the manager that it&#39;s ok
to use transitions here. This means that user could define an animation
that&#39;s played during the transition from one state to the other.

``` csharp
public enum FocusPanelItemState
{
    Normal,
    Unfocused,
    Focused
}
private void OnItemFocused(object sender, EventArgs e)
{
    FocussedPanel = FocussedPanel == sender ? null : sender as FocusPanelItem;
 
    foreach(var item in FocusPanelItems)
    {
        if(item == FocussedPanel)
            item.CurrentState = FocusPanelItemState.Focused;
        else
            item.CurrentState = FocussedPanel == null ? FocusPanelItemState.Normal : FocusPanelItemState.Unfocused;
    }
 
    UpdateItems(true);
}
public FocusPanelItemState CurrentState
{
    get { return currentState; }
    set
    {
        if(currentState == value)
            return;
 
        currentState = value;
 
        VisualStateManager.GoToState(this, currentState.ToString(), true);
    }
}
```

We&#39;re pretty much done, now any one can go ahead and create a new style for our containers and use the VisualStateManager to define a different look for each state. I also altered Generic.xaml to expand the default template to use the states. 

``` xml
<Style TargetType="cx:FocusPanelItem">
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="cx:FocusPanelItem">
                <Border Margin="5,5,5,5" BorderThickness="5" Background="White" CornerRadius="10,10,10,10">
                    <Border.BorderBrush>
                        <LinearGradientBrush EndPoint="0.5,1" StartPoint="0.5,0">
                            <GradientStop x:Name="TopColour" Color="#FF5885A4"/>
                            <GradientStop x:Name="BottomColour" Color="#FF5885A4" Offset="1"/>
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
                    <vsm:VisualStateManager.VisualStateGroups>
                        <vsm:VisualStateGroup x:Name="CommonStates">
                            <vsm:VisualStateGroup.Transitions>
                                <vsm:VisualTransition GeneratedDuration="00:00:00.750" />
                            </vsm:VisualStateGroup.Transitions>
                            <vsm:VisualStateGroup.States>
                                <vsm:VisualState x:Name="Unfocused">
                                    <Storyboard>
                                        <ColorAnimation Storyboard.TargetName="TopColour" Storyboard.TargetProperty="(GradientStop.Color)" To="#005885A4"/>
                                        <ColorAnimation Storyboard.TargetName="BottomColour" Storyboard.TargetProperty="(GradientStop.Color)" To="#FF5885A4"/>
                                    </Storyboard>
                                </vsm:VisualState>
                                <vsm:VisualState x:Name="Focused">
                                    <Storyboard>
                                        <ColorAnimation Storyboard.TargetName="TopColour" Storyboard.TargetProperty="(GradientStop.Color)" To="#FF5885A4"/>
                                        <ColorAnimation Storyboard.TargetName="BottomColour" Storyboard.TargetProperty="(GradientStop.Color)" To="#005885A4"/>
                                    </Storyboard>
                                </vsm:VisualState>
                                <vsm:VisualState x:Name="Normal">
                                    <Storyboard>
                                        <ColorAnimation Storyboard.TargetName="TopColour" Storyboard.TargetProperty="(GradientStop.Color)" To="#FF5885A4"/>
                                        <ColorAnimation Storyboard.TargetName="BottomColour" Storyboard.TargetProperty="(GradientStop.Color)" To="#FF5885A4"/>
                                    </Storyboard>
                                </vsm:VisualState>
                            </vsm:VisualStateGroup.States>
                        </vsm:VisualStateGroup>
                    </vsm:VisualStateManager.VisualStateGroups>
                </Border>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

You can see the final result at &quot;[Focus Panel examples](/examples/controls)&quot; and download the code at &quot;[Focus Panel example code](/content/downloads/compiledexperience.focuspanel.visualstatemanager.zip)&quot;.

