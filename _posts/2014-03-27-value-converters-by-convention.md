---
layout: post
title: Using value converters by convention
tags: caliburn.micro windows-apps windows-phone silverlight wpf
---

The application of a convention in [Caliburn.Micro][cm] is a bit of a pipeline with multiple steps, almost all of them we can modify, let's look at bringing value converters into the equation.

Most of the time the default conventions tend to bring like-minded types together. That's natural in that conventions should be reasonably intuitive, like a string property on the view model being bound to the Text property of the TextBox.

Sometimes these types don't always line up, which is where value converters typically come in. The most common of these being converting between boolean and visibility properties.

Caliburn.Micro does a little bit of work here already, if a convention calls for properties of the two above types (boolean and visibility) then it automatically adds a BooleanToVisibilityConverter to the Binding.

We can extend this through the modification of the [ConventionManager.ApplyValueConverters][conv] method, but why would we do this?
In an app I'm currently working on I want to use relative dates through the app, I'm using a value converter that leverages the [Humanizer][hm] library (check it out if you haven't already) to do the conversion. This means everywhere I want to out put the date I need to dispense with the normal convention using x:Name and set up the binding with the value converter. A bit of a pain and there's always a chance I could miss one.

What we need to do is modify Caliburn such that when it applies a convention between a DateTime (or DateTimeOffset) property and a TextBlock's Text when add the converter to the process.

The code is pretty simple, the only weird part is taking a reference to the default implementation first so that we can keep the default behavior as well.

``` csharp
var baseApplyValueConverter = ConventionManager.ApplyValueConverter;

ConventionManager.ApplyValueConverter = (binding, bindableProperty, property) =>
{
    baseApplyValueConverter(binding, bindableProperty, property);

    if (bindableProperty == TextBlock.TextProperty && typeof(DateTime).IsAssignableFrom(property.PropertyType))
        binding.Converter = new RelativeDateTimeConverter();

    if (bindableProperty == TextBlock.TextProperty && typeof(DateTimeOffset).IsAssignableFrom(property.PropertyType))
        binding.Converter = new RelativeDateTimeConverter();
};
```

Now whenever a convention on a TextBlock is wired to a DateTime it will display a relative date time. We can remove all our bindings and converters from code and go back to simple x:Name conventions.

[cm]: https://github.com/BlueSpire/Caliburn.Micro
[rdt]: http://compiledexperience.com/blog/posts/relative-date-time-converter
[conv]: https://github.com/BlueSpire/Caliburn.Micro/blob/master/src/Caliburn.Micro.Platform/ConventionManager.cs
[hm]: https://github.com/MehdiK/Humanizer
