---
layout: post
title: Adding functionality to Caliburn.Micro conventions
tags: caliburn-micro
---

The binding conventions in Caliburn.Micro can be extended to do almost anything you want. The method `ConventionManager.AddElementConvention<T>` returns a `ElementConvention` which can be further customised. In this post I'll show what things we can do with this by modifying the `ElementConvention.ApplyBinding` action.

Internally Caliburn.Micro already modifies some of the conventions with extra functionality.  It's what turns

``` xml
<TabControl x:Name="Items" />
```

into

``` xml
<TabControl ItemsSource="{Binding Items}" SelectedItem="{Binding ActiveItem, Mode=TwoWay}">
    <TabControl.ContentTemplate>
        <DataTemplate>
            <ContentControl cal:View.Model="{Binding}" VerticalContentAlignment="Stretch" HorizontalContentAlignment="Stretch" IsTabStop="False" />
        </DataTemplate>
    </TabControl.ContentTemplate>
    <TabControl.HeaderTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding DisplayName}" />
        </DataTemplate>
    </TabControl.HeaderTemplate>
</TabControl>
```

We can add our own extensions as well. In this example I'll demonstrate how we can modify the convention for `TextBox` to take into account the `System.ComponentModel.DataAnnotations` attributes.

Let's imagine we have a view model with `FirstName` property that looks like the one below, it would be great if we can by convention apply these attributes to the control the property is bound to. This should let us define in one place all the metadata about this property.

``` csharp
[StringLength(50)]
[Display(Name = "First Name", Prompt = "John")]
public string FirstName
{
    get { return firstName; }
    set
    {
        firstName = value;
        NotifyOfPropertyChange();
    }
}
```

This turns out to be really easy, we simply have something like the following in our app startup code.

``` csharp
ConventionManager.AddElementConvention<TextBox>(TextBox.TextProperty, "Text", "TextChanged")
  .ApplyBinding = (viewModelType, path, property, element, convention) =>
  {
      if (!ConventionManager.SetBindingWithoutBindingOverwrite(viewModelType, path, property, element, convention, TextBox.TextProperty))
      {
          return false;
      }

      var textBox = (TextBox) element;
      var display = property.GetAttributes<DisplayAttribute>(true).FirstOrDefault();
      var length = property.GetAttributes<StringLengthAttribute>(true).FirstOrDefault();

      if (display != null)
      {
          textBox.Header = display.GetName();
          textBox.PlaceholderText = display.GetPrompt();
      }

      if (length != null)
      {
          textBox.MaxLength = length.MaximumLength;
      }

      return true;
  };
```

IF we try to apply the binding between the view model and the `Text` property on the control. If this fails (there's already a binding) then we exit out. We then check the property for the attributes in question and if they exist apply their values to the `TextBox`.

We don't need to make any modifications to the xaml so it can be very simple and still have most of the values it needs applied by the convention itself.

``` xml
<TextBox x:Name="FirstName" />
```

Feel free to experiement in customising your conventions. I hope this gives you all some ideas on what you can do. Let me know what you find.
