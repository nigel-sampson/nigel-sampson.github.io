---
layout: post
title: Using Caliburn Micro as a Data Template Selector
tags: windows-apps windows-phone wpf silverlight caliburn.micro
---

<span class="alignleft"><img src="/content/images/posts/templates-examples.png"/></span>

WPF has a really nice feature in the [DataTemplateSelector][dts] that helps us use different DataTemplate's within the same ItemsControl depending on the data being bound. Out of the box this functionality isn't included with Windows Phone, but it can be implemented in a [number of different ways][wp7dts]. In this post I'm going to show how we can use [Caliburn Micro][cm] to achieve the same functionality.

[Caliburn Micro][cm] is really optimised and designed for a "View Model first" approach, by this we pass the framework the view model we wish to display, it then uses conventions to determine the view for that view model and binds them together.

Windows Phone 7 and it's requirements around the navigation frame and pages enforces a "View first" approach, this doesn't mean we can't use the "View Model" first within our pages. 

When Caliburn applies it's conventions to a ItemsControl (the base class of controls such as ListBox and Pivot) checks to see if the ItemTemplate has been set. If one hasn't Caliburn will set a very simple but powerful DataTemplate that looks like:

``` xml
<DataTemplate>
    <ContentControl cal:View.Model="{Binding}" VerticalContentAlignment="Stretch" HorizontalContentAlignment="Stretch" IsTabStop="False" />
</DataTemplate>
```


By binding the View Model to the Content Control tells Caliburn to locate the appropriate View and instantiate it within the Control Control. The important thing to think about here is that if the view model type is different for items in the ListBox then the views created by Caliburn will be different.

For our example we'll have an activity stream with three different sorts of activities, announcements, status updates and new image galleries.

First we'll create Caliburn View Models for each of our three activities (AnnouncementViewModel, StatusViewModel, GalleryViewModel), we'll have a shared base View Model for common data (ActivityViewModel). On our activity page the view model will expose an observable collection of our base activity view model, on view model initialisation we'll populate the collection with a variety of activities.

``` csharp
public class GalleryViewModel : ActivityViewModel
{
    private string title;
    private Uri image;
 
    public string Title
    {
        get { return title; }
        set
        {
            title = value;
            NotifyOfPropertyChange(() => Title);
        }
    }
 
    public Uri Image
    {
        get { return image; }
        set
        {
            image = value;
            NotifyOfPropertyChange(() => Image);
        }
    }
}
 
public class AnnouncementViewModel : ActivityViewModel
{
    private string headline;
 
    public string Headline
    {
        get { return headline; }
        set
        {
            headline = value;
            NotifyOfPropertyChange(() => Headline);
        }
    }
}
 
public class StatusViewModel : ActivityViewModel
{
    private string username;
    private string text;
 
    public string Username
    {
        get { return username; }
        set
        {
            username = value;
            NotifyOfPropertyChange(() => Username);
        }
    }
 
    public string Text
    {
        get { return text; }
        set
        {
            text = value;
            NotifyOfPropertyChange(() => Text);
        }
    }
}
 
public class ActivityListViewModel : Screen
{
    public ActivityListViewModel()
    {
        Activities = new BindableCollection<ActivityViewModel>();
    }
 
    protected override void OnInitialize()
    {
        Activities.Add(new AnnouncementViewModel
        {
            Headline = "To Do Today 1.5 Released",
            OccuredOn = DateTime.Now
        });
 
        Activities.Add(new GalleryViewModel
        {
            Title = "Lorem Pixum",
            Image = new Uri("http://lorempixum.com/g/140/140/technics/", UriKind.Absolute),
            OccuredOn = DateTime.Today.AddDays(-1)
        });
 
        Activities.Add(new StatusViewModel
        {
            Username = "@nigel-sampson",
            Text = "Up late working on client apps.",
            OccuredOn = new DateTime(2011, 07, 11, 11, 00, 00)
        });
    }
 
    public IObservableCollection<ActivityViewModel> Activities
    {
        get;
        set;
    }
}
```

<span class="alignleft"><img src="/content/images/posts/templates.png"/></span>

Now to create the views for our various activity, rather than the standard Windows Phone view of a PhoneApplicationPage our views should be a UserControl. Remember to follow the conventions Caliburn has so that the view can be located correctly (AnnouncementView, StatusView, GalleryView).

Here's an example of one of the the templates, the AnnouncementView.

``` xml
<UserControl x:Class="CompiledExperience.Phone.Demo.Examples.Views.AnnouncementView"
   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
   xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
   xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
   mc:Ignorable="d"
   FontFamily="{StaticResource PhoneFontFamilyNormal}"
   FontSize="{StaticResource PhoneFontSizeNormal}"
   Foreground="{StaticResource PhoneForegroundBrush}"
   d:DesignHeight="140" d:DesignWidth="456">
 
    <StackPanel x:Name="LayoutRoot" Margin="0,6">
        <TextBlock x:Name="Headline" Style="{StaticResource PhoneTextLargeStyle}"/>
        <StackPanel Orientation="Horizontal">
            <TextBlock Text="occurred on" Style="{StaticResource PhoneTextSubtleStyle}"/>
            <TextBlock x:Name="OccuredOn" Style="{StaticResource PhoneTextSubtleStyle}" Margin="0"/>
        </StackPanel>
    </StackPanel>
</UserControl>
```

Once the views have been created we're pretty much done, by convention Caliburn will bind our Activities list to the ListBox and creates instances of our views based on the view models and we're done.

``` xml
<Grid x:Name="LayoutRoot" Background="Transparent">
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
 
    <StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="12,17,0,28">
        <TextBlock x:Name="ApplicationTitle" Text="COMPILED EXPERIENCE" Style="{StaticResource PhoneTextNormalStyle}"/>
        <TextBlock x:Name="PageTitle" Text="templates" Margin="9,-7,0,0" Style="{StaticResource PhoneTextTitle1Style}"/>
    </StackPanel>
 
    <Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0">
        <ListBox x:Name="Activities"/>
    </Grid>
</Grid>
```

What's really good about this approach is that you don't need to try and build a single data template that needs to deal with all types of content we need to display. Each view can be completely independent of the other views. On a higher level a similar approach can be used to separate the different sections of a Pivot or Panorama.

[cm]: https://github.com/BlueSpire/Caliburn.Micro
[dts]: http://msdn.microsoft.com/en-us/library/ms742521.aspx
[wp7dts]: http://www.windowsphonegeek.com/articles/Implementing-Windows-Phone-7-DataTemplateSelector-and-CustomDataTemplateSelector
