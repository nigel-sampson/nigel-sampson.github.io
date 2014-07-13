---
layout: post
title: Using and abusing CallerMemberName
tags: csharp windows-apps wpf
---

C# 5 came with the [Caller Information attributes][caller], which are ways to mark up your code such that at compile time the values for these method parameters would be populated with the member name, file path and line number of the caller. The canonical use case for this a logging or tracing method like in the [documentation][logging].

The next obvious place is the property changed notifications that form the basis of binding in all the Microsoft Xaml frameworks. Pete Brown provides a good example of this with [Using CallerMemberName for Property Change Notification in Xaml Apps][inpc].

So what’s some other ways we can use this? For application settings I often find myself creating a lot of properties that are simply strong wrappers around the platforms settings dictionary. You end up wiring in the key for the dictionary in a couple of places there’s a lot of [ceremony over essence][ceremony].

``` csharp
public class AppSettingsService
{
    protected static readonly ApplicationDataContainer LocalSettings = ApplicationData.Current.LocalSettings;
 
    public string Username
    {
        get
        {
            if (!LocalSettings.Values.ContainsKey("Username"))
                return String.Empty;
 
            return (string) LocalSettings.Values["Username"];
        }
        set
        {
            LocalSettings.Values["Username"] = value;
        }
    }
}
```

If we create generic Get and Set methods for these dictionaries we can use CallerMemberName as the key and have it automatically filled out by the compiler. To handle this I’ve created a JsonSettingsService for WinRT, it automatically handles serializing the types to json using Json.NET because the underlying container handles only simple types.

``` csharp
public class JsonSettingsService
{
    protected static readonly ApplicationDataContainer LocalSettings = ApplicationData.Current.LocalSettings;
 
    protected static T Get<T>(T defaultValue, [CallerMemberName] string key = "")
    {
        if (!LocalSettings.Values.ContainsKey(key))
            return defaultValue;
 
        var json = (string) LocalSettings.Values[key];
 
        return JsonConvert.DeserializeObject<T>(json);
    }
 
    protected static void Set<T>(T value, [CallerMemberName] string key = "")
    {
        var json = JsonConvert.SerializeObject(value);
 
        LocalSettings.Values[key] = json;
    }
}
```

The Get method takes a default value and the key (annotated with CallerMemberName attribute), while the Set has the value to store and the key. Nothing particularly complicated here.

The usage is where it gets interesting, for my app I’ve created my settings service inheriting from JsonSetttingService, for this example I want to store the details of the current user exposed through our string property with the default value being an empty string. Now the property is simply the following, technically the defaultValue: label isn’t required but I feel it adds some readability to the code.

``` csharp
public class AppSettingsService : JsonSettingsService
{
    public string Username
    {
        get
        {
            return Get(defaultValue: String.Empty);
        }
        set
        {
            Set(value);
        }
    }
}
```

I hope this gives you some ideas on some other ways you use the caller member information to your advantage.

[caller]: http://msdn.microsoft.com/en-us/library/hh534540.aspx
[logging]: http://msdn.microsoft.com/en-us/library/hh534540.aspx
[inpc]: http://10rem.net/blog/2013/02/25/using-callermembername-for-property-change-notification-in-xaml-apps
[ceremony]: http://bryanpendleton.blogspot.co.nz/2010/02/ceremony-vs-essence.html
