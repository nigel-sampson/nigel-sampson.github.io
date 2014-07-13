---
layout: post
title: Using Resource Dictionarys to clean up Generic.xaml
tags: silverlight
---

If you&#39;re ever done any sizable amount of control development in
Silverlight 2 then you&#39;ll know that managing Generic.xaml can be a real
pain. A couple of complicated controls including things like the Visual
State Manager means it won&#39;t be long before you&#39;re looking at hundreds
of lines of xaml that&#39;s not easily managed from Blend.

Thankfully
in Silverlight 3 we have Merged Resource Dictionaries, they can
certainly help organise the styles, templates and other resources in
your application, but they can also help break up Generic.xaml into
more management chunks. In this example we&#39;ll be break up the
FocusPanel we [created](/blog/posts/styling-the-focus-panel-visual-state-manager.aspx) some months ago.

In
Visual Studio you can add a new Resource Dictionary from the &quot;Add New
Item&quot; under the Silverlight category. Now you can copy the styles out
of Generic.xaml into the new Resource Dictionary, create as many as you
want, I tend to organise as it as a Resource Dictionary per control
(which may have a few styles for sub controls). *Important:* Make
sure your new Resource Dictionary has a Build Action of &quot;Resource&quot;, you
can also clear the Custom Tool property if you want.

Now that we have our new Resource Dictionary we need to merge it into Generic.xaml. The syntax is exactly the same as using **ResourceDictionary.MergedDictionaries**
in your normal Silverlight application, the only gotcha is the Uri
format, make sure to include the assembly name in the Uri as follows.

``` xml
<ResourceDictionary
   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
    <ResourceDictionary.MergedDictionaries>
        <ResourceDictionary Source="/CompiledExperience.Core.Controls;component/Themes/FocusPanel.xaml" />
        <ResourceDictionary Source="/CompiledExperience.Core.Controls;component/Themes/ContentPanel.xaml" />
        <ResourceDictionary Source="/CompiledExperience.Core.Controls;component/Themes/Upload.xaml" />
        <ResourceDictionary Source="/CompiledExperience.Core.Controls;component/Themes/ServiceActivity.xaml" />
    </ResourceDictionary.MergedDictionaries>
</ResourceDictionary>
```
Hope this helps someone.

