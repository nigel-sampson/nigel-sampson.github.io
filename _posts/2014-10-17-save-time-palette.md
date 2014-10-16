---
layout: post
title: Save time creating your application palette
tags: windows-apps windows phone
---

A few months ago I wrote a post on [building a colour palette for your app][build-palette], one thing I recommended is creating both the colour and brush resources for each colour separately, like so:

``` xml
<Color x:Key="PlaceholderColor">#FFEBEBEB</Color>
<SolidColorBrush x:Key="PlaceholderBrush" Color="{StaticResource PlaceholderColor}" />
```

Large apps can contain a lot of colours once you start including all the shades and tints of your primary colours and creating all these resources can a bit of a tiresome task.

Thankfully we can make use of regular expressions (gasp!) to save some time in Visual Studio. 

 1. Create all your colour resources
 2. Copy and paste them below the originals and select the copies
 2. Open Visual Studio's **Replace in Files** (Ctrl + Shift + F for me)
 3. Under **Find options** select Use Regular Expressions
 3. Set **Look in:** to Selection
 4. Set **Find what** to `<Color x:Key="(?<name>\w+)Color">#\w+</Color>`
 5. Set **Replace with** to `<SolidColorBrush x:Key="${name}Brush" Color="{StaticResource ${name}Color}"/>`
 6. Hit **Replace All**

Hopefully this can save you a bit of time building your app or at least teach you what you can do with Visual Studio to save yourself some time.

[build-palette]: /blog/posts/colour-palette/