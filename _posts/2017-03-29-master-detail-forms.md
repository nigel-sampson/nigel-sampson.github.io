---
layout: post
title: Supporting Xamarin.Forms Master Detail Page in Caliburn.Micro
tags: caliburn-micro xamarin
---

Previously I've talked about [Supporting Xamarin.Forms Tabbed Page in Caliburn.Micro][tabs] using conductors and Caliburns excellent view location features. I'd like to now show how we can do the same with the `MasterDetailPage`.

The Master / Details pattern is very common in mobile apps providing where a master list of items is presented to the user and when one is select the view is switched to more details.

When using `MasterDetailPage` with `Caliburn.Micro` we want to hit a number of goals.

1. Our main view shouldn't deal with displaying either the master or details and just handle creating the shell of the page.
2. Our main view model should should follow a similar goal only coordinating between master and details.
3. Handle creating two different views for one view model.

In this sample our master / detail item will be characters from the upcoming TV show American Gods. Each character will have a single view model.

``` csharp
public class CharacterViewModel : PropertyChangedBase
{
    public CharacterViewModel(string name, string tagline, string image)
    {
        Name = name;
        Tagline = tagline;
        Image = image;
    }

    public string Name { get; }
    public string Tagline { get; }
    public string Image { get; }
}
```

We'll then create our `ShellViewModel`, it will need to contain a collection of our characters as well as the currently selected character. We'll also need a property that indicicate whether the master menu is currently available. A first draft looks something like:

``` csharp
public class ShellViewModel : Screen
{
    private CharacterViewModel selectedCharacter;
    private bool masterListAvailable;

    public ShellViewModel()
    {
        MasterListAvailable = true;
        Characters = new BindableCollection<CharacterViewModel>
        {
            new CharacterViewModel("Shadow Moon", "The Ex-Con", "character1.jpg"),
            ...
            new CharacterViewModel("Easter", "The Godess of Spring", "character10.jpg"),
        };
    }
    public BindableCollection<CharacterViewModel> Characters { get; }

    public CharacterViewModel SelectedCharacter
    {
        get { return selectedCharacter; }
        set
        {
            selectedCharacter = value;
            NotifyOfPropertyChange();
            MasterListAvailable = SelectedCharacter == null;
        }
    }
    public bool MasterListAvailable
    {
        get { return masterListAvailable; }
        set
        {
            masterListAvailable = value;
            NotifyOfPropertyChange();
        }
    }
}
```

The most interesting things in here are populating the data in the constuctor, really this is only things specific to characters in the entire view model, but could easily be abstracted away. The other is that when the `SelectedCharacter` is set we update `MasterListAvailable` depending on the value. What this will do ultimately in the UI is that when a character is selected in the master list the UI will switch over to the details view.

We already have a conductor in the framework that can handle this `Conductor<T>.Collection.OneActive` leaving us with.

``` csharp
 public class ShellViewModel : Conductor<CharacterViewModel>.Collection.OneActive
 {
     private bool masterListAvailable;

     public ShellViewModel()
     {
         MasterListAvailable = true;

         Items.AddRange(new []
         {
             new CharacterViewModel("Shadow Moon", "The Ex-Con", "character1.jpg"),
             ...
             new CharacterViewModel("Easter", "The Godess of Spring", "character10.jpg"),
         });
     }

     protected override void OnActivationProcessed(CharacterViewModel item, bool success)
     {
         MasterListAvailable = item == null;
     }

     public bool MasterListAvailable
     {
         get { return masterListAvailable; }
         set
         {
             masterListAvailable = value;
             NotifyOfPropertyChange();
         }
     }
 }
```

Our `ShellView` comes next, an important point to note here is that no where in the view do we declare how a character should be display we simple style the master list and the container for the detail content. This view could be reused across any master / details view in the application.

``` xml
<?xml version="1.0" encoding="utf-8" ?>
<MasterDetailPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:views="clr-namespace:MasterDetail.Views;assembly=MasterDetail"
             xmlns:cm="clr-namespace:Caliburn.Micro.Xamarin.Forms;assembly=Caliburn.Micro.Platform.Xamarin.Forms"
             x:Class="MasterDetail.Views.ShellView"
             IsPresented="{Binding MasterListAvailable}">
  <MasterDetailPage.Master>
    <ContentPage Title="Master">
      <ListView ItemsSource="{Binding Items}" SelectedItem="{Binding ActiveItem, Mode=TwoWay}">
        <ListView.ItemTemplate>
          <DataTemplate>
            <ViewCell>
              <ContentView cm:View.Model="{Binding}" cm:View.Context="MasterView" />
            </ViewCell>
          </DataTemplate>
        </ListView.ItemTemplate>
      </ListView>
    </ContentPage>
  </MasterDetailPage.Master>
  <MasterDetailPage.Detail>
    <ContentPage cm:View.Model="{Binding ActiveItem}" cm:View.Context="DetailView" Title="Master" />
  </MasterDetailPage.Detail>
</MasterDetailPage>
```

The imporatant part to note here is we're using `View.Model` in the same way as our [last post about TabbedPage][tabs]. Normally this would be since both the master data template and details content page are both being bound to a `CharacterViewModel` then a `CharacterView` could be injected.

This wouldn't be desirable since we don't want the same view for both places. Instead we use `View.Content` to provide some extra view location.

For the master list item we provide a context of `MasterView`, this means when resolving the view for `MasterDetail.ViewModels.CharacterViewModel` the result will not be the typical `MasterDetail.Views.CharacterView` but `MasterDetail.Views.Character.MasterView` and `MasterDetail.Views.Character.DetaisView` for our details panel. We now have two views for the same view model based on their context which can significantly reduce the complexity of our view models.

The contents of the master and details view are straight forward.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<ContentView xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:Class="MasterDetail.Views.Character.MasterView">
  <ContentView.Content>
    <StackLayout Padding="12">
      <Label Text="{Binding Name}" />
    </StackLayout>
  </ContentView.Content>
</ContentView>
```

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<ContentView xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:Class="MasterDetail.Views.Character.DetailView">
  <ContentView.Content>
    <Grid>
      <Image Source="{Binding Image}" Grid.Row="0" Aspect="AspectFill" />
      <StackLayout Grid.Row="0" Padding="24" BackgroundColor="#66000000">
        <Label Text="{Binding Name}" TextColor="#FFFFFF" FontSize="24" FontAttributes="Bold" HorizontalTextAlignment="Center" />
        <Label Text="{Binding Tagline}" TextColor="#FFFFFF" FontAttributes="Italic" HorizontalTextAlignment="Center" />  
      </StackLayout>
    </Grid>
  </ContentView.Content>
</ContentView>
```

So there we have it, by making use of `View.Model` and `View.Context` we can create conducting view and view models that can be responsible for only the flow between master and details and not be mixed up with the data or display of the items themselves.

We can use contexts to have multiples views for a view model even further reducing this complexity.

The code for all of this is up on a new [GitHub samples repository][samples] where I plan to post more of the code from posts like these.

[tabs]: /blog/posts/tabbed-page-conductor
[sample]: https://github.com/nigel-sampson/samples/tree/master/MasterDetail