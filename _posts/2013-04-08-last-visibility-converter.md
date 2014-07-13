---
layout: post
title: The last Visibility Converter
tags: csharp windows-apps windows-phone wpf silverlight
---

The Boolean to Visibility Converter is as close you’re going to get to bread and butter in the xaml frameworks. You need it in almost every app and almost every framework has one, even the Windows 8 project templates come with one.

The trouble is that a lot of these implementations are naïve and simplistic, and soon enough you’ll be writing “Inverse Boolean to Visibility Converter”, “Int32 to Visibility Converter” etc. Ultimately these all exhibit very similar behavior, convert the default value (false, 0, null etc.) to Collapsed and anything else to Visible or do the same but inversed. Having over a dozen converters for all the implementations becomes a pain to manage and awkward.

So let’s build the last Visibility Converter you’ll need, we’ve already defined what the behavior should be, “Convert the default value to Collapsed and everything else to Visible”. The first thing we need to do is determine the default value for a type. If the type is a value type (int, boolean etc.) then we create an instance of the type otherwise the type is a reference type so the default value is null. Below is an example of this as an extension method for WinRT.

``` csharp
public static object GetDefaultValue(this Type type)
{
    return type.GetTypeInfo().IsValueType ? Activator.CreateInstance(type) : null;
}
```

Now we can create our converter, we’ll want to have two properties to customize the behavior, the first, Inverse is pretty simplistic, the second, SupportIsNullOrEmpty is to deal with the one exception to our rules above and that’s string. The default value for string is null but we’ll typically want to treat an empty string as null, so we’ll add a second property SupportIsNullOrEmpty to be able to turn on or off dealing empty strings as null (it’ll be on by default).

``` csharp
public class VisibilityConverter : IValueConverter
{
    public VisibilityConverter()
    {
        SupportIsNullOrEmpty = true;
    }
 
    public bool Inverse
    {
        get;
        set;
    }
 
    public bool SupportIsNullOrEmpty
    {
        get; set;
    }
 
    public object Convert(object value, Type targetType, object parameter, string language)
    {
        bool visible;
 
        if (value is string && SupportIsNullOrEmpty)
        {
            visible = !String.IsNullOrEmpty(value.ToString());
        }
        else
        {
            var defaultValue = value != null ? value.GetType().GetDefaultValue() : null;
 
            visible = !Equals(value, defaultValue);
        }
 
        if (Inverse)
            visible = !visible;
 
        return visible ? Visibility.Visible : Visibility.Collapsed;
    }
 
    public object ConvertBack(object value, Type targetType, object parameter, string language)
    {
        throw new NotSupportedException();
    }
}
```

The only downside to this approach is that ConvertBack can’t be implemented sensibly but to be honest I’ve never found a reason to have a * to Visibility converter to need it (that’s not to say there aren’t some).

