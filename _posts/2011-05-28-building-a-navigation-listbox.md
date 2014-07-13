---
layout: post
title: Building a Navigation ListBox
tags: csharp windows-phone
---

A common usage pattern for a ListBox is to use it as a navigation list, when you select the item the application navigates to a new page. This usually involves either wiring into the ListBox SelectionChanged event, or using some sort of GestureTrigger.

Why not build the navigation functionality directly into the ListBox itself. Each item bound the NavigationList will expose a property with the uri. Much like the charting controls we'll then supply a Binding to the list that will be used for each item when it's selected.

``` csharp
public class ApplicationViewModel : PropertyChangedBase
{
    private int id;
    private string name;
 
    public ApplicationViewModel(int id, string name)
    {
        this.id = id;
        this.name = name;
    }
 
    public int Id
    {
        get { return id; }
        set
        {
            id = value;
 
            NotifyOfPropertyChange("Id");
            NotifyOfPropertyChange("Uri");
        }
    }
 
    public Uri Uri
    {
        get
        {
            return new Uri("/Views/NavigationListBox/DetailsView.xaml?Id=" + Id, UriKind.Relative);
        }
    }
 
    public string Name
    {
        get { return name; }
        set
        {
            name = value;
 
            NotifyOfPropertyChange("Name");
        }
    }
}
```


``` xml
<toolkit:NavigationListBox x:Name="Apps" NavigateUriBinding="{Binding Uri}">
    <toolkit:NavigationListBox.ItemTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Name}" Style="{StaticResource PhoneTextLargeStyle}"/>
        </DataTemplate>
    </toolkit:NavigationListBox.ItemTemplate>
    <toolkit:NavigationListBox.EmptyContent>
        <TextBlock Text="No Apps" Style="{StaticResource PhoneTextLargeStyle}"/>
    </toolkit:NavigationListBox.EmptyContent>
</toolkit:NavigationListBox>
```

Our NavigationList will obviously inherit from the ListBox we created in the last post. We'll then expose a property NavigateUriBinding, this property is actually of type Binding. It's also important that it's not a DependencyProperty, just a regular property, otherwise when we set the binding in xaml we'll create a binding to the property rather than setting the property to the binding (a slightly confusing concept I know). The best way to explain it is that while we're setting the binding once on the list we'll we be using it on each item in the list. This means that we need to have actual Binding object rather than a DependencyProperty bound to a value.

We'll create an attached dependency property at the same time, this is where we'll be setting the NavigateUri for each ListItem. We'll then override the PrepareContainerForItemOverride, this is where the ListBox creates the ListItem for each item bound, we'll then set the binding of our attached property to the binding set in NavigateUriBinding.

``` csharp
public class NavigationListBox : ListBox
{
    public static readonly DependencyProperty NavigateUriProperty =
        DependencyProperty.RegisterAttached("NavigateUri", typeof(Uri), typeof(NavigationListBox), null);
 
    public NavigationListBox()
    {
        SelectionChanged += OnSelectionChanged;
    }
 
    public Binding NavigateUriBinding
    {
        get; set;
    }
 
    public static Uri GetNavigateUri(DependencyObject obj)
    {
        return (Uri)obj.GetValue(NavigateUriProperty);
    }
 
    public static void SetNavigateUri(DependencyObject obj, Uri value)
    {
        obj.SetValue(NavigateUriProperty, value);
    }
 
    protected override void PrepareContainerForItemOverride(DependencyObject element, object item)
    {
        base.PrepareContainerForItemOverride(element, item);
 
        var frameworkElement = element as FrameworkElement;
 
        if(frameworkElement == null || NavigateUriBinding == null)
            return;
 
        frameworkElement.SetBinding(NavigateUriProperty, NavigateUriBinding);
    }
 
    private void OnSelectionChanged(object sender, SelectionChangedEventArgs e)
    {
        if(SelectedItem == null || ItemContainerGenerator == null)
            return;
 
        var container = ItemContainerGenerator.ContainerFromItem(SelectedItem);
        var uri = GetNavigateUri(container);
 
        SelectedItem = null;
 
        var frame = Application.Current.RootVisual as PhoneApplicationFrame;
 
        if(frame == null || uri == null)
            return;
 
        frame.Navigate(uri);
    }
}
```

There isn't an override for the SelectionChanged event so we'll create a handler in the constructor of the control. If an item has been selected we'll retrieve the value of our attached property and it's not null we'll navigate to that control. Easy!

In a bid to learn Mercurial at the same time I've uploaded the source for the controls so far (with a few other little bits and pieces) to Bitbucket, you find it all at [Compiled Experience Toolkit](https://bitbucket.org/nigel.sampson/compiled-experience-toolkit). Sorry for the delay.

[Download the source](https://bitbucket.org/nigel.sampson/compiled-experience-toolkit).
