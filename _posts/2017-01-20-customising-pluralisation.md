---
layout: post
title: Customising Pluralisation in Caliburn.Micro
tags: caliburn-micro
---

Caliburn.Micro uses pluralisation in it's conventions. It's what turns an `x:Name` of `Products` into bindings of the `Products` and `SelectedProduct` properties.

To be more accurate Caliburn.Micro uses Singularisation, and it's implementation is rather naive that certainly don't cope with the variety the english language provides, let alone other languages.

It's current implementation is:

``` csharp
public static Func<string, string> Singularize = original =>
{
    return original.EndsWith("ies")
        ? original.TrimEnd('s').TrimEnd('e').TrimEnd('i') + "y"
        : original.TrimEnd('s');
};
```

There are however some fanstasic libraries out there to deal with manipulating language in this way, and Caliburn.Micro can be updated to use them. I'm talking in particular about the library [Humanizer][humanizer].

Having the following in our application configure code switches the framework to use Humanizer for it's conventions and be able to deal with all the property names it couldn't deal with before.

``` csharp
ConventionManager.Singularize = original => original.Singularize(inputIsKnownToBePlural: false);
```

[humanizer]: https://github.com/Humanizr/Humanizer
