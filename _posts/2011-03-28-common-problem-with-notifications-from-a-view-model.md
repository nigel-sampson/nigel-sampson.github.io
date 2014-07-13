---
layout: post
title: Common Problem with Notifications from a View Model
tags: csharp windows-phone silverlight wpf
---

In the MVVM (Model, View, ViewModel) pattern in xaml based applications (Windows Phone 7, Silverlight and WPF) there are two different ways a view model will notify the view of changes. If you're unaware of the differences you can end up wondering why your changes aren't being reflected in your view.

The most common is implementing the INotifyPropertyChanged interface, the other is through the INotifyCollectionChanged interface. Most developers really won't use second interface but its useful to know about and how it affects your view model.

When you bind to a property on your view model the binding infrastructure will listen for INotifyPropertyChanged events from your view model for that property. 

If the type of the property implements INotifyCollectionChanged (such as ObservableCollection) it will also listen for events off that property.

The distinction is important because a common pattern with developers is implement collection properties as automatic properties. This however limits the way you can interact with the collection such that changes will be reflected in the view.

``` csharp
public class CorrectViewModel : ViewModelBase
{
    public CorrectViewModel()
    {
        Items = new ObservableCollection<Item>();??
    }
 
    public ObservableCollection<Item> Items
    {
        get; set;
    }
 
    public void Add()
    {
        var items = GetItems();
 
        Items.Clear(); // Collection changed raised
 
        foreach(var item in items)
        {
            Items.Add(item); // Collection changed raised
        }
    }
 
    private IEnumerable<Item> GetItems()
    {
        return from i in Enumerable.Range(0, 10)
                select new Item
                {
                    Name = "Item: " + i
                };
    }
}
 
public class IncorrectViewModel : ViewModelBase
{
    public IncorrectViewModel()
    {
        Items = new ObservableCollection<Item>();
    }
 
    public ObservableCollection<Item> Items
    {
        get;
        set;
    }
 
    public void Add()
    {
        var items = GetItems();
 
        Items = new ObservableCollection<Item>(); // No notification raised
 
        foreach(var item in items)
        {
            // Collection changed raised but view unaware of new collection
            Items.Add(item);
        }
    }
 
    private IEnumerable<Item> GetItems()
    {
        return from i in Enumerable.Range(0, 10)
                select new Item
                {
                    Name = "Item: " + i
                };
    }
}
```

Notice the difference between the correct view model and the incorrect one is that the correct one alters the already existing collection and the view is notified with INotifyCollectionChanged events. The incorrect view model replaces the collection with an entirely new one, but because the automatic property doesn't fire an INotifyPropertyChanged event the changes will not be reflected in the view.

The incorrect view model can be fixed by either modifying the Add method to not replace the collection but modify it in place, or to modify the Items collection to fire property changed events.
