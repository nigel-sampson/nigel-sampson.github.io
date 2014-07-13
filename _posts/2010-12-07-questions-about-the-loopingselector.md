---
layout: post
title: Questions about the LoopingSelector
tags: csharp windows-phone
---

I've had a few questions about using the LoopingSelector described in "[Using LoopingSelector from the Silverlight Toolkit](/blog/posts/using-loopingselector-from-the-silverlight-toolkit)", and I'd like to go over them.

### How do I use this in MVVM?
By default the LoopingSelector doesn't expose a SelectedItem to bind to, instead the ILoopingSelectorDataSource does. The approach I ended up using was to expose my custom Data Source off the View Model and bind that to the DataSource of the LoopingSelector.

This allows the ViewModel to manipulate the data of the data source as well as work with the SelectedItem. Initially I was hesitant about the approach but thinking about it some more I prefer it. It separates the data from the user interface using the View / View Model split and exposes the data in the most appropriate manner. The view and view model would like something like:

``` csharp
public class PageViewModel : ViewModelBase
{
    private ILoopingSelectorDataSource names;
 
    public PageViewModel()
    {
        Names = new LoopingListDataSource<string>(new[] { "Larry", "Curly", "Moe"});
    }
 
    public ILoopingSelectorDataSource Names
    {
        get
        {
            return names;
        }
        set
        {
            names = value;
            OnPropertyChanged("Names");
        }
    }
}
```

``` xml
<controls:LoopingSelector DataSource="{Binding Names}" ItemSize="48, 48" Width="48"/>
```

### How do I use text values instead of numbers?
The beauty of the ILoopingSelectorDataSource is that it's completely data type agnostic, your data source can be anything. I simply used numbers since it was easy to demo, the real key is in the GetNext and GetPrevious methods, this is what determines the display order of the list, if they return strings then you're pretty much working with text values. Here's an example of a list based data source.

``` csharp
public class LoopingListDataSource<T> : ILoopingSelectorDataSource
{
    private T selectedItem;
    private readonly IList<T> items;
 
    public event EventHandler<SelectionChangedEventArgs> SelectionChanged;
 
    public LoopingListDataSource(IEnumerable<T> items)
    {
        this.items = items.ToList();
        selectedItem = this.items.First();
    }
 
    protected virtual void OnSelectionChanged(SelectionChangedEventArgs e)
    {
        var selectionChanged = SelectionChanged;
 
        if(selectionChanged != null)
            selectionChanged(this, e);
    }
 
    public T SelectedItem
    {
        get
        {
            return selectedItem;
        }
        set
        {
            var oldValue = selectedItem;
 
            var newValue = value;
 
            if(oldValue.Equals(newValue))
                return;
 
            selectedItem = newValue;
 
            OnSelectionChanged(new SelectionChangedEventArgs(new[] { oldValue }, new[] { newValue }));
        }
    }
 
    public T GetNext(T relativeTo)
    {
        var index = items.IndexOf(relativeTo) + 1;
 
        return index >= items.Count ? items[0] : items[index];
    }
 
    public T GetPrevious(T relativeTo)
    {
        var index = items.IndexOf(relativeTo) - 1;
 
        return index < 0 ? items[items.Count - 1] : items[index];
    }
 
    object ILoopingSelectorDataSource.GetNext(object relativeTo)
    {
        return GetNext((T)relativeTo);
    }
 
    object ILoopingSelectorDataSource.GetPrevious(object relativeTo)
    {
        return GetPrevious((T)relativeTo);
    }
 
    object ILoopingSelectorDataSource.SelectedItem
    {
        get
        {
            return selectedItem;
        }
        set
        {
            SelectedItem = (T)value;
        }
    }
}
```
