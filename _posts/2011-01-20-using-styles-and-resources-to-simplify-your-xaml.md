---
layout: post
title: Using styles and resources to simplify your xaml
tags: silverlight windows-phone wpf
---

The Windows Phone 7 SDK comes with a lot of great templates, styles and resources to get you up and running quickly. But they can be a little repetitive, notice that every page set's a lot of it's own properties such as Font that will the same over all your pages. When you have a dozen or more pages (which isn't that hard to do) you're going to be repeating a lot of xaml. 

Also if you're taking advantage of the TransitionFrame from the [Silverlight Toolkit](http://silverlight.codeplex.com/) then you also be needing to specify transitions for each page, wouldn't it be better to just define these once and reuse them?

A much better solution is to create a set of styles and resources that build on the existing styles in order simply the xaml in any given page.

In order to simply keep this nice and simple we'll create a separate resource dictionary for our new styles to avoid cluttering App.xaml. The Phone SDK doesn't have an item template for Resource Dictionary, so I end up just selecting new text file and giving it the extension of .xaml. Once created I paste the following code in it to set it up as an empty resource file. 

``` xml
<ResourceDictionary
   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
   xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
   xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
   mc:Ignorable="d">
 
</ResourceDictionary>
```

We then modify App.xaml to include the merged dictionary.

``` xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary Source="Resources/Styles.xaml"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

Here's the default Style I use for my pages, these can be overridden in the page itself so you can set a few things that may have exceptions later on. 

``` xml
<Style x:Key="DefaultPage" TargetType="phone:PhoneApplicationPage">
    <Setter Property="FontFamily" Value="{StaticResource PhoneFontFamilyNormal}"/>
    <Setter Property="FontSize" Value="{StaticResource PhoneFontSizeNormal}"/>
    <Setter Property="Foreground" Value="{StaticResource PhoneForegroundBrush}"/>
    <Setter Property="SupportedOrientations" Value="PortraitOrLandscape"/>
    <Setter Property="Orientation" Value="Portrait"/>
    <Setter Property="shell:SystemTray.IsVisible" Value="True"/>
    <Setter Property="toolkit:TransitionService.NavigationInTransition">
        <Setter.Value>
            <toolkit:NavigationInTransition>
                <toolkit:NavigationInTransition.Backward>
                    <toolkit:TurnstileTransition Mode="BackwardIn"/>
                </toolkit:NavigationInTransition.Backward>
                <toolkit:NavigationInTransition.Forward>
                    <toolkit:TurnstileTransition Mode="ForwardIn"/>
                </toolkit:NavigationInTransition.Forward>
            </toolkit:NavigationInTransition>
        </Setter.Value>
    </Setter>
    <Setter Property="toolkit:TransitionService.NavigationOutTransition">
        <Setter.Value>
            <toolkit:NavigationOutTransition>
                <toolkit:NavigationOutTransition.Backward>
                    <toolkit:TurnstileTransition Mode="BackwardOut"/>
                </toolkit:NavigationOutTransition.Backward>
                <toolkit:NavigationOutTransition.Forward>
                    <toolkit:TurnstileTransition Mode="ForwardOut"/>
                </toolkit:NavigationOutTransition.Forward>
            </toolkit:NavigationOutTransition>
        </Setter.Value>
    </Setter>
</Style>
```

We can apply that any page we create now and strip out a lot of the repetitive styling code.

```
Style="{StaticResource DefaultPage}"
```

So what other things can we push into styles and resources? Pretty much anything we want. A few examples from [To Do Today](/windows-phone/to-do) that are resources include.

 - Application name, resources can be strings.
 - Data templates for List Pickers
 - Value converters
 - Style extensions for things such as form labels.
