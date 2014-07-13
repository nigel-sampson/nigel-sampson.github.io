---
layout: post
title: Metro Theme Resources and Templates
tags: windows-apps
---

One thing that always helps when building xaml applications is to have easy access to reference the Theme resources and control Templates.

Thankfully for building Windows 8 Metro style applications you can find the appropriate files at  **C:\Program Files\Windows Kits\8.0\Include\winrt\xaml\design**. 

**ThemeResources.xaml** contains the resource dictionaries for the Dark (Default), Light and HighContrast themes while **Generic.xaml** contains the default styles and templates for most of the xaml controls. Incredibly useful when needing to know how the control is put together and how you can affect it.

For instance FlipView has a 3 pixel margin built into it, which is nice to know before trying to track down why your alignment is a little in your own code.

I'm spending a lot of time with Windows 8 at the moment so will be blogging about it more here or a the [Marker Metro][mm] website.

[mm]: http://markermetro.com
