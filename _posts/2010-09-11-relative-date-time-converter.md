---
layout: post
title: Relative Date Time Converter
tags: csharp silverlight wpf windows-phone
---

Value converters are really what help make good applications become great, they plug into the binding infrastructure and help convert values appropriate to the model into values appropriate for the view. Being able to convert say a temperature from a simple integer into something like a SolidColourBrush gives a very easy representation of heat.

These work in all three of the xaml based frameworks, Windows Phone 7, Silverlight and WPF and because value converters are so compact they're also very testable.

A lot of sites represent days in a more interesting "5 minutes ago" format, so I set about throwing together a quick value converter for this, converting from a date to a string. I've become a big fan of using dictionaries of functions in this way for times when many if statements become too messy and a strategy style pattern is overkill.

``` csharp
public class RelativeDateTimeConverter : IValueConverter
{
    private const int Minute = 60;
    private const int Hour = Minute * 60;
    private const int Day = Hour * 24;
    private const int Year = Day * 365;
 
    private readonly Dictionary<long, Func<TimeSpan, string>> thresholds = new Dictionary<long, Func<TimeSpan, string>>
    {
        {2, t => "a second ago"},
        {Minute,  t => String.Format("{0} seconds ago", (int)t.TotalSeconds)},
        {Minute * 2,  t => "a minute ago"},
        {Hour,  t => String.Format("{0} minutes ago", (int)t.TotalMinutes)},
        {Hour * 2,  t => "an hour ago"},
        {Day,  t => String.Format("{0} hours ago", (int)t.TotalHours)},
        {Day * 2,  t => "yesterday"},
        {Day * 30,  t => String.Format("{0} days ago", (int)t.TotalDays)},
        {Day * 60,  t => "last month"},
        {Year,  t => String.Format("{0} months ago", (int)t.TotalDays / 30)},
        {Year * 2,  t => "last year"},
        {Int64.MaxValue,  t => String.Format("{0} years ago", (int)t.TotalDays / 365)}
    };
 
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        var dateTime = (DateTime)value;
        var difference = DateTime.UtcNow - dateTime.ToUniversalTime();
 
        return thresholds.First(t => difference.TotalSeconds < t.Key).Value(difference);
    }
 
    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
    {
        throw new NotSupportedException();
    }
}
```

``` xml
<TextBlock Style="{StaticResource PhoneTextSubtleStyle}" Text="{Binding Created, Converter={StaticResource RelativeDateTimeConverter}}" />
```
