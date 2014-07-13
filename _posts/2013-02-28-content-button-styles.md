---
layout: post
title: Content button styles
tags: windows-apps windows-phone
---

When building apps sometimes you need to make things “ click-able” with either a mouse, finger or some other pointing type device. The default button templates for both Windows Phone and Windows 8 are great, but aren't useful if you just want to wrap them around some content.

Possible solutions I've seen include not using buttons but just using something like the Tapped event on content, others including re-styling the Content Template of the button to remove everything but the content itself.

Personally I think there’s a middle ground, keeping some of the visual states for the button can be handy, we still would like to give the user feedback that the button can be pressed through a tilt effect. Also still having the ability to add a border or background can be useful. I use these styles a lot, they work well in list boxes, about pages and creating fake “tiles”.

To that end here’s the Windows 8 and Windows Phone styles for Content button. As well as an example of how to add some extra styling to create a Tile button.

###Windows Phone

``` xml
<Style x:Key="ContentButton" TargetType="Button">
    <Setter Property="toolkit:TiltEffect.IsTiltEnabled" Value="True"/>
    <Setter Property="HorizontalAlignment" Value="Left"/>
    <Setter Property="Background" Value="Transparent"/>
    <Setter Property="BorderBrush" Value="{StaticResource PhoneForegroundBrush}"/>
    <Setter Property="Foreground" Value="{StaticResource PhoneForegroundBrush}"/>
    <Setter Property="BorderThickness" Value="0"/>
    <Setter Property="FontFamily" Value="{StaticResource PhoneFontFamilySemiBold}"/>
    <Setter Property="FontSize" Value="{StaticResource PhoneFontSizeMedium}"/>
    <Setter Property="Margin" Value="0" />
    <Setter Property="Padding" Value="0" />
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="ButtonBase">
                <Grid Background="Transparent">
                    <VisualStateManager.VisualStateGroups>
                        <VisualStateGroup x:Name="CommonStates">
                            <VisualState x:Name="Normal"/>
                            <VisualState x:Name="MouseOver"/>
                            <VisualState x:Name="Pressed"/>
                            <VisualState x:Name="Disabled">
                                <Storyboard>
                                    <ObjectAnimationUsingKeyFrames Storyboard.TargetName="ContentContainer" Storyboard.TargetProperty="Opacity">
                                        <DiscreteObjectKeyFrame KeyTime="0" Value="0.5" />
                                    </ObjectAnimationUsingKeyFrames>
                                </Storyboard>
                            </VisualState>
                        </VisualStateGroup>
                    </VisualStateManager.VisualStateGroups>
                    <Border x:Name="ButtonBackground" BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}" CornerRadius="0" Background="{TemplateBinding Background}" Margin="0" >
                        <ContentControl x:Name="ContentContainer" Foreground="{TemplateBinding Foreground}" HorizontalContentAlignment="{TemplateBinding HorizontalContentAlignment}" VerticalContentAlignment="{TemplateBinding VerticalContentAlignment}" Padding="{TemplateBinding Padding}" Content="{TemplateBinding Content}" ContentTemplate="{TemplateBinding ContentTemplate}"/>
                    </Border>
                </Grid>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
```

###Windows 8

``` xml
<Style x:Key="ContentButton" TargetType="Button">
    <Setter Property="Background" Value="{StaticResource ButtonBackgroundThemeBrush}" />
    <Setter Property="Foreground" Value="{StaticResource ButtonForegroundThemeBrush}"/>
    <Setter Property="BorderBrush" Value="{StaticResource ButtonBorderThemeBrush}" />
    <Setter Property="BorderThickness" Value="0" />
    <Setter Property="Padding" Value="0" />
    <Setter Property="HorizontalAlignment" Value="Left" />
    <Setter Property="VerticalAlignment" Value="Top" />
    <Setter Property="HorizontalContentAlignment" Value="Stretch" />
    <Setter Property="VerticalContentAlignment" Value="Stretch" />
    <Setter Property="FontFamily" Value="{StaticResource ContentControlThemeFontFamily}" />
    <Setter Property="FontWeight" Value="SemiBold" />
    <Setter Property="FontSize" Value="{StaticResource ControlContentThemeFontSize}" />
    <Setter Property="Template">
        <Setter.Value>
            <ControlTemplate TargetType="Button">
                <Grid>
                    <VisualStateManager.VisualStateGroups>
                        <VisualStateGroup x:Name="CommonStates">
                            <VisualState x:Name="Normal" />
                            <VisualState x:Name="PointerOver" />
                            <VisualState x:Name="Pressed">
                                <Storyboard>
                                    <PointerDownThemeAnimation TargetName="Border"/>
                                </Storyboard>
                            </VisualState>
                            <VisualState x:Name="Disabled" />
                        </VisualStateGroup>
                        <VisualStateGroup x:Name="FocusStates">
                            <VisualState x:Name="Focused">
                                <Storyboard>
                                    <DoubleAnimation Storyboard.TargetName="FocusVisualWhite"
                                                    Storyboard.TargetProperty="Opacity"
                                                    To="1"
                                                    Duration="0" />
                                    <DoubleAnimation Storyboard.TargetName="FocusVisualBlack"
                                                    Storyboard.TargetProperty="Opacity"
                                                    To="1"
                                                    Duration="0" />
                                </Storyboard>
                            </VisualState>
                            <VisualState x:Name="Unfocused" />
                            <VisualState x:Name="PointerFocused" />
                        </VisualStateGroup>
                    </VisualStateManager.VisualStateGroups>
                    <Border x:Name="Border"
                           Background="{TemplateBinding Background}"
                           BorderBrush="{TemplateBinding BorderBrush}"
                           BorderThickness="{TemplateBinding BorderThickness}"
                           Margin="0">
                        <ContentPresenter x:Name="ContentPresenter"
                                         Content="{TemplateBinding Content}"
                                         ContentTransitions="{TemplateBinding ContentTransitions}"
                                         ContentTemplate="{TemplateBinding ContentTemplate}"
                                         Margin="{TemplateBinding Padding}"
                                         HorizontalAlignment="{TemplateBinding HorizontalContentAlignment}"
                                         VerticalAlignment="{TemplateBinding VerticalContentAlignment}" />
                    </Border>
                    <Rectangle x:Name="FocusVisualWhite"
                              IsHitTestVisible="False"
                              Stroke="{StaticResource FocusVisualWhiteStrokeThemeBrush}"
                              StrokeEndLineCap="Square"
                              StrokeDashArray="1,1"
                              Opacity="0"
                              StrokeDashOffset="1.5" />
                    <Rectangle x:Name="FocusVisualBlack"
                              IsHitTestVisible="False"
                              Stroke="{StaticResource FocusVisualBlackStrokeThemeBrush}"
                              StrokeEndLineCap="Square"
                              StrokeDashArray="1,1"
                              Opacity="0"
                              StrokeDashOffset="0.5" />
                </Grid>
            </ControlTemplate>
        </Setter.Value>
    </Setter>
</Style>
 
<Style x:Key="TileButton" TargetType="Button" BasedOn="{StaticResource ContentButton}">
    <Setter Property="Width" Value="140"/>
    <Setter Property="Height" Value="140"/>
    <Setter Property="Background" Value="{StaticResource AccentBrush}"/>
    <Setter Property="Padding" Value="10" />
    <Setter Property="VerticalContentAlignment" Value="Bottom" />
    <Setter Property="FontSize" Value="26.667"/>
    <Setter Property="FontWeight" Value="Light"/>
    <Setter Property="Margin" Value="0,0,20,20"/>
    <Setter Property="Foreground" Value="{StaticResource ListViewItemOverlayForegroundThemeBrush}" />
</Style>
```
