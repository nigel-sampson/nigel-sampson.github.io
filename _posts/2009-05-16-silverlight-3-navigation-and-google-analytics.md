---
layout: post
title: Silverlight 3 Navigation and Google Analytics
tags: csharp silverlight
---

The navigation framework in Silverlight 3 is a welcome addition as it reduces a lot of plumbing code required to start building multi-page Silverlight applications. One thing that multi-page Silverlight applications need is analytics.

Under traditional analytics frameworks a user visiting a Silverlight application will only render as a visitor with a single page view. If nothing else this will skew the bounce rate of your analytics no matter how many pages they actually visit. Lets fix this.

The first thing we're going to need is a way to track Silverlight pages into Google Analytics, I threw together a simple class call the Analytics code via the HtmlBridge, this code assumes the tracking code as been placed on the page hosting the Silverlight application.

``` csharp
public class GoogleAnalyticsTracker
{
    private readonly ScriptObject tracker;
 
    public GoogleAnalyticsTracker(string accountCode)
    {
        if(!HtmlPage.IsEnabled)
            return;
 
        var gat = (ScriptObject)HtmlPage.Window.GetProperty("_gat");
 
        if(gat == null)
            throw new InvalidOperationException("Could not locate the property '_gat'. Has the Google Analytics script been included on the page?");
 
        tracker = (ScriptObject)gat.Invoke("_getTracker", accountCode);
 
        if(tracker == null)
            return;
 
        tracker.Invoke("_setAllowAnchor", true);
    }
 
    public void TrackPageView(Uri uri)
    {
        if(tracker == null)
            return;
 
        var documentUri = HtmlPage.Document.DocumentUri;
 
        var trackingUri = documentUri.AbsolutePath + uri + documentUri.Query;
 
        MessageBox.Show(String.Format("Tracking {0}", trackingUri));
 
        tracker.Invoke("_trackPageview", trackingUri);
    }
}
```

One gotcha is that it looks like calling _trackPageview will ignore anything after the '#' character in the url so we need to rearrange the url a little. Slightly annoying because it means you can't go directly from analytics to the page in Silverlight in question. The other gotacha is that because we're using the Html bridge to call the Analytics code we can't use this in Out of Browser applications.

Once we have that it's a simple process of creating the tracking code in the MainWindow and attaching to the Navigated event of the Frame, which then calls the tracking code like so.

``` csharp
public partial class MainPage
{
    private readonly GoogleAnalyticsTracker tracker = new GoogleAnalyticsTracker("UA-xxxxxxx-1");
 
    public MainPage()
    {
        InitializeComponent();
 
        Frame.Navigated += OnFrameNavigated;
    }
 
    private void OnFrameNavigated(object sender, NavigationEventArgs e)
    {
        tracker.TrackPageView(e.Uri);
    }
 
    private void NavButton_Click(object sender, RoutedEventArgs e)
    {
        var navigationButton = (Button)sender;
        var goToPage = navigationButton.Tag.ToString();
 
        Frame.Navigate(new Uri(goToPage, UriKind.Relative));
    }
}
```

As you can see we're now getting our Silverlight pages showing up in Google Analytics as page views correctly.

![Google Analytics](/content/images/posts/analytics.png)
