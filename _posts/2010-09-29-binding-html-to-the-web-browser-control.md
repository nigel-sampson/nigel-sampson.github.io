---
layout: post
title: Binding Html to the Web Browser Control
tags: csharp silverlight windows-phone
---

Currently the fastest way to display a string of html on the Windows Phone 7 is to use the Web Browser control, simply call the *NavigateToString* method passing the html you want to display.

When trying to use this in an MVVM pattern the method call becomes slightly problematic, we'd much rather have a bindable property. Thankfully we can use attached properties to achieve this.

``` csharp
public static class WebBrowserHelper
{
    public static readonly DependencyProperty HtmlProperty = DependencyProperty.RegisterAttached(
        "Html", typeof(string), typeof(WebBrowserHelper), new PropertyMetadata(OnHtmlChanged));
 
    public static string GetHtml(DependencyObject dependencyObject)
    {
        return (string)dependencyObject.GetValue(HtmlProperty);
    }
 
    public static void SetHtml(DependencyObject dependencyObject, string value)
    {
        dependencyObject.SetValue(HtmlProperty, value);
    }
 
    private static void OnHtmlChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        var browser = d as WebBrowser;
 
        if(browser == null)
            return;
 
        var html = e.NewValue.ToString();
 
        browser.NavigateToString(html);
    }
}
```

Once you have the attached property the binding becomes very simple.

``` xml
<phone:WebBrowser cxi:WebBrowserHelper.Html="{Binding Question.Body}" />
```

If you really want to add some polish to your app I suggest looking at fellow kiwi Ben Gracewood's solution to having your html respect the current phone theme at [Integrated Links and Styling for Windows Phone 7 Web Browser Control](http://www.ben.geek.nz/2010/07/integrated-links-and-styling-for-windows-phone-7-webbrowser-control/).

