---
layout: post
title: What's new in Windows 10 - C# 6.0
tags: windows-apps windows-phone csharp
---

Microsoft have finally started to show off what's available in Windows 10 for us app developers. Over the next few days we can start to take a look at some of these changes and what it can mean. Today we'll look at what it means now we can develop with C# 6.

## Disclaimer

This was written using the initial release of the Windows 10 Developer Tools running on Windows 10 build 10041 and things will mostly change before final release. I'll try to come back and keep these posts up to date as time goes on.

## C# 6

While this isn't technically a Windows 10 feature it's something that will kind of slip under the radar. When we build Windows 10 apps with Visual Studio 2015 we'll have the C# 6 features available to us. There's  huge list of new features, and I don't want to cover all of them just highlight a few that show up in Windows 10 / MVVM apps.

### Null conditional operators

The null conditional operator reduces the amount of code you're going to need to write for null checking. Because the feature evaluates the left hand side of the `?.` only once it provides a thread safe way to invoke events like follows.

``` csharp
protected void OnPropertyChanged(string propertyName)
{
	PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
}
```

### nameof operator

The `nameof` operator is a simple one that provides a a string that names a program element, obvious use cases are for things like `ArgumentNullException`. However property changed notifications are another, anything that means less strings in your code is a good thing in my books.

``` csharp
public string Username
{
	get { return username; }
	set
	{
		username = value;
		OnPropertyChanged(nameof(Username));
	}
}
```

## Conclusion

There's a lot of really cool features coming in C# 6, one of the best places to check them out is the the [C# team blog](http://blogs.msdn.com/b/csharpfaq/archive/2014/11/20/new-features-in-c-6.aspx). I'd suggest thinking about how they'll affect your codebase and don't forget you have them available to you when building Windows 10 apps.

