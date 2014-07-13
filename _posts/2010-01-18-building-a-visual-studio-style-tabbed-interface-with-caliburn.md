---
layout: post
title: Building a Visual Studio Style Tabbed Interface with Caliburn
tags: csharp silverlight wpf caliburn
---

In the application I'm currently working I'm using an interface patten similar to Visual Studio with multiple tabs (work items) open at any one time. [Caliburn](http://caliburn.codeplex.com/) has some great utiltity baked in for this style of application with their [IPresenter Component Model](http://caliburn.codeplex.com/wikipage?title=IPresenter%20Component%20Model&amp;referringTitle=Documentation).&nbsp; Ultimately we'll be building something that looks like:

![Visual Studio Tabs](/content/images/posts/tabs.png)

*Note: There was an [issue](http://caliburn.codeplex.com/WorkItem/View.aspx?WorkItemId=4865) in the v1 RTW that caused the following example to function incorrectly. The 1.1 branch in the Codeplex Repository correct this.*

While this post is mainly targeting WPF a lot of it (minus the ContextMenu) can be used in Silverlight as well.

I'm going to assume you have some experience with setting up Caliburn and won't dig too much into that if not I'd start at the [Caliburn documentation](http://caliburn.codeplex.com/documentation). By default Caliburn wants to use IServiceLocator to create the child presenters, I've created a simple IViewModelFactory to allow me to plug extra functionality into this process, for this article the implementation is as follows.

``` csharp
public class DefaultViewModelFactory : IViewModelFactory
{
    private readonly IServiceLocator serviceLocator;
 
    public DefaultViewModelFactory(IServiceLocator serviceLocator)
    {
        if(serviceLocator == null)
            throw new ArgumentNullException("serviceLocator");
 
        this.serviceLocator = serviceLocator;
    }
 
    public T Create<T>() where T : IPresenter
    {
        return serviceLocator.GetInstance<T>();
    }
}
```

We'll start with building the ViewModel for our application, we'll name it WorkspaceViewModel and because we'll be dealing with multiple "child" presenters and the idea of a current presenter we'll inherit this class from MultiPresenterManager. After that we'll add some methods to manage the child presenters, specially CloseThis, CloseAll and CloseAllButThis. We'll also create a dummy method to open a sample "child view", more like the WorkspaceViewModel will be responding to events from an event aggregator to open child views.

``` csharp
public class WorkspaceViewModel : MultiPresenterManager
{
    private readonly IViewModelFactory viewModelFactory;
 
    public WorkspaceViewModel(IViewModelFactory viewModelFactory)
    {
        this.viewModelFactory = viewModelFactory;
    }
 
    public void OpenSimpleViewModel()
    {
        var viewModel = viewModelFactory.Create<SimpleViewModel>();
 
        this.Open(viewModel);
    }
 
    public void CloseAll()
    {
        foreach(Presenter presenter in Presenters.ToList())
        {
            presenter.Close();
        }
    }
 
    public void ClosePresenter(Presenter selectedPresenter)
    {
        selectedPresenter.Close();
    }
 
    public void CloseAllButThis(Presenter selectedPresenter)
    {
        foreach(Presenter presenter in Presenters.ToList())
        {
            if(presenter == selectedPresenter)
                continue;
 
            presenter.Close();
        }
    }
}
```

Now we have the ViewModel we need to build the corresponding view. To we'll need a couple of things to begin with, obviously a TabControl to display our child presenters and little button to open new presenters.

``` xml
<TabControl Grid.Row="1" x:Name="Presenters" ItemsSource="{Binding Presenters}" SelectedItem="{Binding CurrentPresenter}">
    <TabControl.ItemTemplate>
        <DataTemplate>
            <Grid>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*" />
                </Grid.ColumnDefinitions>
                <TextBlock Text="{Binding DisplayName}" />
            </Grid>
        </DataTemplate>
    </TabControl.ItemTemplate>
    <TabControl.ContentTemplate>
        <DataTemplate>
            <ContentControl caliburn:View.Model="{Binding}" />
        </DataTemplate>
    </TabControl.ContentTemplate>
</TabControl>
```

This should be enough to get started, the button is attached to the OpenPresenter and when clicked the Presenters collection is changed which opens another tab in the TabControl.

``` xml
<Button Margin="4" Content="Open" caliburn:Message.Attach="OpenSimpleViewModel"/>
```

Now lets hook up the familar context menu on the tabs, we'll need to do a bit of wrangling with the Message Target in Caliburn as we want to target the actual presenter and not the current DataContext (in the TabItem it's the child presenter and not the workspace presenter). The simplest way to do this is to bind to something outside the tab control. We can then attach messages for CloseThis, CloseAll and CloseAllButThis to each menu item.

``` xml
<UserControl.Resources>
    <FrameworkElement x:Key="DataContextReference"/>
</UserControl.Resources>
<UserControl.DataContext>
    <Binding Mode="OneWayToSource" Path="DataContext" Source="{StaticResource DataContextReference}"/>
</UserControl.DataContext>

<ContextMenu caliburn:Action.TargetWithoutContext="{Binding DataContext, Source={StaticResource DataContextReference}}">
    <MenuItem Header="Close" caliburn:Message.Attach="ClosePresenter($dataContext)" />
    <MenuItem Header="Close All But This" caliburn:Message.Attach="CloseAllButThis($dataContext)" />
    <MenuItem Header="Close All" caliburn:Message.Attach="CloseAll" />
</ContextMenu>
```

Now the only thing we've left to do is recreate the the drop down and close box in the top right corner. The combobox is bound to the same values as the tab control and the button is attached to the CloseThis method using the currently selected tab as the parameter.

``` xml
<Button Margin="4" Content="x" caliburn:Message.Attach="ClosePresenter(Presenters.SelectedItem)"/>
<ComboBox ItemsSource="{Binding Presenters}" SelectedItem="{Binding CurrentPresenter}" Margin="4" DisplayMemberPath="DisplayName"/>
```
The full xaml for the view is as follows.

``` xml
<UserControl
   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:caliburn="http://www.caliburnproject.org"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    x:Class="CompiledExperience.Azure.Explorer.Client.Shell.Views.WorkspaceView"
   mc:Ignorable="d" d:DesignWidth="503.507" d:DesignHeight="384.96"
   >
    <UserControl.Resources>
        <FrameworkElement x:Key="DataContextReference"/>
    </UserControl.Resources>
    <UserControl.DataContext>
        <Binding Mode="OneWayToSource" Path="DataContext" Source="{StaticResource DataContextReference}"/>
    </UserControl.DataContext>
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition/>
        </Grid.RowDefinitions>
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Left">
            <Button Margin="4" Content="Open" caliburn:Message.Attach="OpenSimpleViewModel"/>
            <Button Margin="4" Content="x" caliburn:Message.Attach="ClosePresenter(Presenters.SelectedItem)"/>
            <ComboBox ItemsSource="{Binding Presenters}" SelectedItem="{Binding CurrentPresenter}" Margin="4" DisplayMemberPath="DisplayName"/>
        </StackPanel>
        <TabControl Grid.Row="1" x:Name="Presenters" ItemsSource="{Binding Presenters}" SelectedItem="{Binding CurrentPresenter}">
            <TabControl.ItemTemplate>
                <DataTemplate>
                    <Grid>
                        <Grid.ContextMenu>
                            <ContextMenu caliburn:Action.TargetWithoutContext="{Binding DataContext, Source={StaticResource DataContextReference}}">
                                <MenuItem Header="Close" caliburn:Message.Attach="ClosePresenter($dataContext)" />
                                <MenuItem Header="Close All But This" caliburn:Message.Attach="CloseAllButThis($dataContext)" />
                                <MenuItem Header="Close All" caliburn:Message.Attach="CloseAll" />
                            </ContextMenu>
                        </Grid.ContextMenu>
                        <Grid.ColumnDefinitions>
                            <ColumnDefinition Width="*" />
                        </Grid.ColumnDefinitions>
                        <TextBlock Text="{Binding DisplayName}" />
                    </Grid>
                </DataTemplate>
            </TabControl.ItemTemplate>
            <TabControl.ContentTemplate>
                <DataTemplate>
                    <ContentControl caliburn:View.Model="{Binding}" />
                </DataTemplate>
            </TabControl.ContentTemplate>
        </TabControl>
    </Grid>
</UserControl>
```

There's a good [Stack Overflow](http://stackoverflow.com/questions/561931/how-to-create-trapezoid-tabs-in-wpf-tab-control) question about styling the tab control like Visual Studio so I'll leave to that to someone with actual design skills but from here you're pretty much complete. Some stuff you'll most likely want to extend it with is dealing with duplicate tabs, you may not want to have two options tabs open and so on.
