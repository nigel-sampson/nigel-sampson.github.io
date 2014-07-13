---
layout: post
title: Useful Value Converters
tags: csharp silverlight wpf windows-phone
---

Value converters are a really useful part of the xaml binding infrastructure, they work in Windows Phone 7, Silverlight and WPF. As we work on more and more projects in this space we build up a library of useful value converters. I'd like to illustrate some of the ones I use here.

### Bitmap Image Converter
``` csharp
public class BitmapImageConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if(value is string)
            return new BitmapImage(new Uri((string)value, UriKind.RelativeOrAbsolute));
 
        if(value is Uri)
            return new BitmapImage((Uri)value);
 
        throw new NotSupportedException();
    }
 
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotSupportedException();
    }
}
```

### Boolean To Visibility Converter
``` csharp
public class BooleanToVisibilityConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if(value == null)
            return Visibility.Collapsed;
 
        var isVisible = (bool)value;
 
        return isVisible ? Visibility.Visible : Visibility.Collapsed;
    }
 
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var visiblity = (Visibility)value;
 
        return visiblity == Visibility.Visible;
    }
}
```

### Colour To Brush Converter
``` csharp
public class ColorToBrushConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if(value == null)
            return new SolidColorBrush(Color.FromArgb(0, 0, 0, 0));
 
        if(value is Color)
            return new SolidColorBrush((Color)value);
 
        if(value is string)
            return new SolidColorBrush(Parse((string)value));
 
        throw new NotSupportedException("ColorToBurshConverter only supports converting from Color and String");
    }
 
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotSupportedException();
    }
 
    private static Color Parse(string color)
    {
        var offset = color.StartsWith("#") ? 1 : 0;
 
        var a = Byte.Parse(color.Substring(0 + offset, 2), NumberStyles.HexNumber);
        var r = Byte.Parse(color.Substring(2 + offset, 2), NumberStyles.HexNumber);
        var g = Byte.Parse(color.Substring(4 + offset, 2), NumberStyles.HexNumber);
        var b = Byte.Parse(color.Substring(6 + offset, 2), NumberStyles.HexNumber);
 
        return Color.FromArgb(a, r, g, b);
    }
}
```

### Simple Type Converter
``` csharp
public class SimpleTypeConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return System.Convert.ChangeType(value, targetType, CultureInfo.CurrentCulture);
    }
 
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        return System.Convert.ChangeType(value, targetType, CultureInfo.CurrentCulture);
    }
}
```

### String Format Converter
``` csharp
public class StringFormatConverter : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        if(parameter == null && value == null)
            return String.Empty;
 
        if(parameter == null)
            return value.ToString();
 
        return String.Format(CultureInfo.CurrentCulture, parameter.ToString(), value);
    }
 
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotSupportedException();
    }
}
```


One nice thing about value converters is because they're simple pluggable pieces of code they've easy to unit test. The unit tests for our Boolean To Visibility Converter may look like.

``` csharp
[TestClass]
public class BooleanToVisibilityConverterFixture
{
    [TestMethod]
    public void ConvertWithNullValueReturnsCollapsed()
    {
        var converter = new BooleanToVisibilityConverter();
        var value = converter.Convert(null, typeof(Visibility), null, CultureInfo.CurrentCulture);
 
        Assert.AreEqual(Visibility.Collapsed, value);
    }
 
    [TestMethod]
    public void ConvertWithFalseReturnsCollapsed()
    {
        var converter = new BooleanToVisibilityConverter();
        var value = converter.Convert(false, typeof(Visibility), null, CultureInfo.CurrentCulture);
 
        Assert.AreEqual(Visibility.Collapsed, value);
    }
 
    [TestMethod]
    public void ConvertWithTrueReturnsVisible()
    {
        var converter = new BooleanToVisibilityConverter();
        var value = converter.Convert(true, typeof(Visibility), null, CultureInfo.CurrentCulture);
 
        Assert.AreEqual(Visibility.Visible, value);
    }
 
    [TestMethod]
    public void ConvertBackWithVisibleReturnsTrue()
    {
        var converter = new BooleanToVisibilityConverter();
        var value = converter.ConvertBack(Visibility.Visible, typeof(bool), null, CultureInfo.CurrentCulture);
 
        Assert.AreEqual(true, value);
    }
 
    [TestMethod]
    public void ConvertBackWithCollapsedReturnsFalse()
    {
        var converter = new BooleanToVisibilityConverter();
        var value = converter.ConvertBack(Visibility.Collapsed, typeof(bool), null, CultureInfo.CurrentCulture);
 
        Assert.AreEqual(false, value);
    }
}
```

I tend to create a resource dictionary named Converters.xaml to hold references to all the converters I need.

``` xml
<ResourceDictionary
   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
   xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
   xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
   xmlns:c="clr-namespace:CompiledExperience.Kore.Client.Converters"
   mc:Ignorable="d">
 
    <c:BitmapImageConverter x:Key="BitmapImage"/>
    <c:BooleanToVisibilityConverter x:Key="BooleanToVisiblity"/>
    <c:ColorToBrushConverter x:Key="ColorToBrush"/>
    <c:SimpleTypeConverter x:Key="SimpleType"/>
    <c:StringFormatConverter x:Key="StringFormat"/>
 
</ResourceDictionary>
```

Once that's created then you modify your Binding statements to use the Converter.

``` xml
<Rectangle Fill="{Binding Colour, Converter={StaticResource ColourToBrushConverter}}" Width="12" Height="72" />
```

If you're using Expression Blend to wire up your bindings then you can select the Converter from the provided drop down.
