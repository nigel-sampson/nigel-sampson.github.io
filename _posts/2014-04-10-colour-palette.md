---
layout: post
title: Building a colour palette for your app
tags: windows-apps windows-phone
---

When creating your app on both the phone and tablet almost all [guidelines][msdn] will tell you create a consistent colour palette. There's a lot of tools out there to help you design your palette, [Kuler][kuler] from Adobe is a good example. But what's the best way to use our palette in your app?

There's a couple of goals here, the first is we only want to have to define our colours once for the entire app. This way if we need to tweak them or do a wholesale replacement we're not spending hours hunting for every instance of that colour.

The second is to provide an easy way to override existing theme resources.

What I tend to do is create a separate resource dictionary for the palette, this is because it's usually imported into other dictionaries and I want to keep the repetition low. The other resource dictionary I'll create at this point is a theme overrides one, this contains all the redefinition of the built in resources using the new palette of colours.

In our palette dictionary we'll want to define the colours and brushes separately, this is because despite most of the time we'll be using the brush there will be times the colour is required. Naturally the brush definition refers to the colour so we're sticking with our goals. Typically a colour / brush definition will look like:

``` xml
<Color x:Key="MajorAccentColor">#FF2980B9</Color>
<SolidColorBrush x:Key="MajorAccentBrush" Color="{StaticResource MajorAccentColor}"/>
```

Sometimes your designs may make use of the same colour but with different alpha (or opacity) values. For me if this brush is going to be used through the app I'll define it within the palette like so:

``` xml
<Color x:Key="MinorAccentColor">#FF80B929</Color>
<SolidColorBrush x:Key="MinorAccentBrush" Color="{StaticResource MinorAccentColor}"/>
<SolidColorBrush x:Key="MinorAccentOverlayBrush" Color="{StaticResource MinorAccentColor}" Opacity="0.5"/>
```

However if it's a one off I'll probably define the brush as inline XAML referring back to our colour resource.

``` xml
<controls:Sidebar x:Name="Sidebar" IsExpandedChanged="OnIsExpandedChanged">
    <controls:Sidebar.Background>
        <SolidColorBrush Color="{StaticResource SidebarBackgroundColor}" Opacity="0.5"/>
    </controls:Sidebar.Background>
```

I try to name the colour / brush pairs in generic names more along the lines of their usage than what they look like colour wise. This is because they'll often be tweaked and it can look rather silly to have a resource named "RedColour" that's actually blue.

Now our palette is defined let's make use of it, the first thing I go through and do is redefine quite a few of the out of the box theme resources to make the controls more in line with the rest of my app. It's these little bits that really polish an app, I die a little on the inside when I see the default purple in combo boxes and progress rings. I swear Microsoft chose that colour so people would be forced to deal with it but a lot don't.

In my theme overrides resource dictionary I import the palette and start to define styles for the controls in terms of colours from the palette. We don't need to redefine every single resource, typically only the ones that add the accent colour. For instance with the ComboBox control the overrides are:

``` xml
<ResourceDictionary.MergedDictionaries>
    <ResourceDictionary Source="/Resources/Palette.xaml"/>
</ResourceDictionary.MergedDictionaries>
```

``` xml
<SolidColorBrush x:Key="ComboBoxItemSelectedBackgroundThemeBrush" Color="{StaticResource MinorAccentColor}" />
<SolidColorBrush x:Key="ComboBoxItemSelectedPointerOverBackgroundThemeBrush" Color="{StaticResource LightMinorAccentColor}" />
<SolidColorBrush x:Key="ComboBoxSelectedBackgroundThemeBrush" Color="{StaticResource MinorAccentColor}" />
<SolidColorBrush x:Key="ComboBoxSelectedPointerOverBackgroundThemeBrush" Color="{StaticResource LightMinorAccentColor}" />
```

That's pretty much it, a nice easy way to define your colour scheme realistically in any XAML app and reuse it well. 

[msdn]: http://msdn.microsoft.com/library/windows/apps/dn439319.aspx
[kuler]: https://kuler.adobe.com/
