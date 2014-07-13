---
layout: post
title: Caliburn Micro conventions for Windows Phone
tags: csharp caliburn.micro windows-phone
---

One of the best things I find in the MVVM framework [Caliburn Micro](http://caliburnmicro.codeplex.com/) is the convention based binding for properties and methods. They allow a lot of binding between your view and view model to happen "auto-magically" based on the same of elements in your view.

If you haven't read the documentation I highly recommend reading [All about Conventions](http://caliburnmicro.codeplex.com/wikipage?title=All%20About%20Conventions&referringTitle=Documentation) as it explains a lot of the internal plumbing that makes the conventions work.

You can add more conventions really easily through the ConventionManager class, these are usually set up in the Configure method of your Bootstrapper. For instance if we wanted to add the standard convention for buttons we'd use the following code.

``` csharp
AddElementConvention<ButtonBase>(ButtonBase.ContentProperty, "DataContext", "Click");
```

The first parameter is the property on the element to bind to if we match against a property on the view model. For our button example if we had a string property on the ViewModel it would be the text of the Button.

The second is the property to use if we refer to the element as a parameter of an action, currently this isn't possible in Windows Phone at the moment (this looks to be changing in Mango but I haven't tested it yet) so can be ignored for now.

The third is the name of the event to trigger any actions specified on the element or for ones matched by convention, for our Button it makes sense that the Click event should trigger the action.

One other nice part of the ConventionManager in Caliburn is that you can provide conventions for base types and any that inherit from that type will also inherit it's conventions. This makes writing up all the conventions for a library considerably easier and that some of the base conventions in Caliburn will already suffice.

By default Caliburn has a lot of conventions built in, you can see most of them in the article linked above. What it's missing are conventions for more of the phone only controls, including the [Silverlight Toolkit](http://silverlight.codeplex.com/). This makes sense as we shouldn't expect to have Caliburn depend upon those assemblies just for some conventions.

I thought I'd provide some of the conventions I've used in projects recently.

``` csharp
// Phone Controls (from the Caliburn Sample
ConventionManager.AddElementConvention<Pivot>(ItemsControl.ItemsSourceProperty, "SelectedItem", "SelectionChanged").ApplyBinding =
    (viewModelType, path, property, element, convention) =>
    {
        if(ConventionManager.GetElementConvention(typeof(ItemsControl)).ApplyBinding(viewModelType, path, property, element, convention))
        {
            ConventionManager.ConfigureSelectedItem(element, Pivot.SelectedItemProperty, viewModelType, path);
            ConventionManager.ApplyHeaderTemplate(element, Pivot.HeaderTemplateProperty, viewModelType);
 
            return true;
        }
 
        return false;
    };
 
ConventionManager.AddElementConvention<Panorama>(ItemsControl.ItemsSourceProperty, "SelectedItem", "SelectionChanged").ApplyBinding =
    (viewModelType, path, property, element, convention) =>
    {
        if(ConventionManager.GetElementConvention(typeof(ItemsControl)).ApplyBinding(viewModelType, path, property, element, convention))
        {
            ConventionManager.ConfigureSelectedItem(element, Panorama.SelectedItemProperty, viewModelType, path);
            ConventionManager.ApplyHeaderTemplate(element, Panorama.HeaderTemplateProperty, viewModelType);
 
            return true;
        }
 
        return false;
    };
 
// Silverlight Toolkit
ConventionManager.AddElementConvention<AutoCompleteBox>(AutoCompleteBox.ItemsSourceProperty, "SelectedItem", "SelectionChanged").ApplyBinding =
    (viewModelType, path, property, element, convention) =>
    {
        ConventionManager.ConfigureSelectedItem(element, AutoCompleteBox.SelectedItemProperty, viewModelType, path);
 
        return true;
    };
ConventionManager.AddElementConvention<DateTimePickerBase>(DateTimePickerBase.ValueProperty, "Value", "ValueChanged");

ConventionManager.AddElementConvention<PerformanceProgressBar>(PerformanceProgressBar.IsIndeterminateProperty, "IsIndeterminate", "Loaded");

ConventionManager.AddElementConvention<ToggleSwitch>(ToggleSwitch.IsCheckedProperty, "IsChecked", "Checked");

ConventionManager.AddElementConvention<MenuItem>(ItemsControl.ItemsSourceProperty, "DataContext", "Click");
 
// Maps Conventions
ConventionManager.AddElementConvention<MapItemsControl>(ItemsControl.ItemsSourceProperty, "DataContext", "Loaded");

ConventionManager.AddElementConvention<Pushpin>(ContentControl.ContentProperty, "DataContext", "MouseLeftButtonDown");
```
