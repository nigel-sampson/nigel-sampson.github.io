---
layout: post
title: Supporting Xamarin.Forms Tabbed Page in Caliburn.Micro
tags: caliburn-micro xamarin
---

One of the best features of Caliburn.Micro in my opinion is get away from "one view model per screen" and starting to compose view models together to form the screen.

The framework itself supplies conductors to help facilitate this, as well as the behaviour of `View.Model` as as "Template Selector" on steroids.

As a quick refresher what `View.Model` does when applied to `ContentControl` (or `ContentView` in Xamarin.Forms) is locate the view appropriate to the bound view model and inject it into the control (while binding the view and view model together).

For instance given the following xaml:

``` xml
<Grid>
    <ContentControl cm:View.Model="{Binding ActiveItem}" />
</Grid>
```

The contents of the `ContentControl` will be the view appropriate for the type of `ActiveItem`. If for instance it was `ProductsListViewModel` then the contents would be `ProductsListView`. What's really great is that as `ActiveItem` changes to am instance of another type then the contents wil update with the new view as well. 

So how does this fit with `TabbedPage` in Xamarin.Forms?

Well it works pretty seamlessly with a few tricks.

First we'll start with the view model that represents the tabbed page itself, I typically call this the shell. It inherits from `Conductor<PageViewModel>.Collection.OneActive` which essentially means that this view model conducts a collection of page view models where one is active at a time. This maps nicely to how a tab UI works.

``` csharp
public class ShellViewModel : Conductor<PageViewModel>.Collection.OneActive
{
    ...
}
```

In our shell view we have the `TabbedPage` itself, bound to the `Items` and `ActiveItem` of the shell view model. For the item template we have one gotcha. `TabbedPage` expects it's contents to inherit from `Page`, so the root element of our data template but be `ContentPage`. Fortunately `View.Model` can handle this with no problem. We also make use of the display name property on `Screen` to supply the title for each tab.

``` xml
<TabbedPage
    ItemsSource="{Binding Items}"
    SelectedItem="{Binding ActiveItem, Mode=TwoWay}">
    <TabbedPage.ItemTemplate>
        <DataTemplate>
            <ContentPage Title="{Binding DisplayName}" cm:View.Model="{Binding}" />
        </DataTemplate>
    </TabbedPage.ItemTemplate>
</TabbedPage
```

In our shell we can now add some pages:

``` csharp
protected override void OnInitialize()
{
    Items.Add(new ProductsListViewModel());
    Items.Add(new LocationsViewModel());
    Items.Add(new ContactViewModel());

    ActivateItem(Items[0]);
}
```

As the tabs are selected our item template will locate the correct view for the above view models.

What's really interesting about this is that our shell doesn't have to care about how the pages below it look or work. It can focus on it's single reponsiblity which is the tabs themselves and any behaviour conducting their flow.