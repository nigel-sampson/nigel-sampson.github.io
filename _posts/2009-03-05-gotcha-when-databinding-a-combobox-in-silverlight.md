---
layout: post
title: Gotcha when databinding a ComboBox in Silverlight
tags: csharp silverlight
---

Occasionally we all run into bugs in systems not built by ourselves.
Some are minor, some however really make you wonder how much QA was
really done. Sadly this was the latter.

I found this one when
building a system that bound a collection of objects from a service to
a ComboBox, at one point this collection changes and the list is
refreshed from the service. I had code in this refresh to set the
selected item (bound using a two way data binding) to the previously
selected item. It was failing with the falling exception when I
attempted to set the SelectedItem.

```
Error: Unhandled Error in Silverlight 2 Application 
Code: 4004    
Category: ManagedRuntimeError       
Message: System.ArgumentException: Value does not fall within the expected range.
   at MS.Internal.XcpImports.MethodEx(IntPtr ptr, String name, CValue[] cvData)
   at MS.Internal.XcpImports.MethodPack(IntPtr objectPtr, String methodName, Object[] rawData)
   at MS.Internal.XcpImports.UIElement_TransformToVisual(UIElement element, UIElement visual)
   at System.Windows.UIElement.TransformToVisual(UIElement visual)
   at System.Windows.Controls.Primitives.Selector.IsOnCurrentPage(Int32 index, Rect& itemsHostRect, Rect& listBoxItemRect)
   at System.Windows.Controls.Primitives.Selector.ScrollIntoView(Int32 index)
   at System.Windows.Controls.Primitives.Selector.SetFocusedItem(Int32 index, Boolean scrollIntoView)
   at System.Windows.Controls.ComboBox.PrepareContainerForItemOverride(DependencyObject element, Object item)
   at System.Windows.Controls.ItemsControl.UpdateContainerForItem(Int32 index)
   at System.Windows.Controls.ItemsControl.RecreateVisualChildren()
   at System.Windows.Controls.ItemsControl.RecreateVisualChildren(IntPtr unmanagedObj)    
```

Assuming a page with a ComboBox and a Button this is the simplest piece of code that recretes the error. Just change the ComboBox to any item other than the first and then hit the Refresh button. 

``` csharp
public partial class Page
{
    public Page()
    {
        InitializeComponent();
        Loaded += OnLoaded;
        RefreshButton.Click += OnRefresh;
    }
 
    private void OnLoaded(object sender, RoutedEventArgs e)
    {
        BindVideos();
    }
 
    private void BindVideos()
    {
        var videos = GetVideos();
        VideoList.ItemsSource = videos;
        VideoList.SelectedItem = videos[0];
    }
 
    private void OnRefresh(object sender, RoutedEventArgs e)
    {
        BindVideos();
    }
 
    private static ObservableCollection<Video> GetVideos()
    {
        return new ObservableCollection<Video>
        {
            new Video(1, "test.wmv"),
            new Video(2, "tv-ad.wmv"),
            new Video(3, "porn.wmv")
        };
    }
}
```

There&#39;s a few possible solutions out there, the one that seemed to work for me was calling ComboBox.UpdateLayout like so.

``` csharp
private void BindVideos()
{
    var videos = GetVideos();
    VideoList.ItemsSource = videos;
    VideoList.UpdateLayout();
    VideoList.SelectedItem = videos[0];
}
```

While
this is a workable solution for the example I&#39;m trying to build my
solution using the Model View ViewModel (MVVM) pattern. This makes the
solution useless since my ViewModel does the interaction. I tried a
solution where my View listened for PropertyChanges on the ViewModel
but since you can&#39;t guarantee in which order these events occur it
wasn&#39;t viable. In the end I created a new event on my ViewModel that
the View listens to and calls UpdateLayout. It&#39;s not a nice solution
and I feel dirty for having to use it.

This needs to be fixed. It&#39;s hours of my life I won&#39;t be getting back.

Edit : I&#39;m glad to say it&#39;s fixed in [Silverlight 3](/blog/posts/combo-box-data-binding-in-silverlight-3).

