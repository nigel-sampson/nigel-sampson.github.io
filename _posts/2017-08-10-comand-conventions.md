---
layout: post
title: Command conventions in Caliburn.Micro
tags: csharp xamarin caliburn-micro
---

If you've ever spoken to me personally at a convention or a user group (I'll be at [NDC Sydney][ndc] if next week, comes say hello) then you may have heard me talk about disliking command objects in MVVM. 

In my opinion (and it's just that, my opinion) most commands don't add any value to the main goals for using MVVM (maintainability, readability and testability). Typically they're just an object that wraps a method, sometimes also a predicate for CanExecute but that's it. What they mostly add is ceremony to the view model and not much more, you typically see this in commands named RelayCommand or DelegateCommand.

Some commands are really useful however, ReactiveCommand in [ReactiveUI][rxui] adds a lot of value when building that style of application and I highly recommend them.

How can we customize the conventions in Caliburn.Micro to make use of commands? Given that we'd be binding to the command property on the control we can simply modify the convention for controls such as Button. 

This change to the convention is set up during app initialisation. The important parameter is the first, this defines that when we find a property on the view model matching the `x:Name` of the button then this is the dependency property we'll bind that property to.

``` csharp
ConventionManager.AddElementConvention<Button>(ButtonBase.CommandProperty, "CommandParameter", "Click");
```

To keep our view model sample simple I'll use a `DelegateCommand`

``` csharp
public LoginViewModel()
{
    Login = new DelegateCommand<string>(LoginImpl, CanLogin);
}

public ICommand Login { get; }

private bool CanLogin(string token) => !String.IsNullOrWhiteSpace(token);

private void LoginImpl(string token)
{
   .... 
}
```

Now with our convention in place we can simply give the `Button` the `x:Name` we normally would but instead of a method being called the command will be executed.

``` xml
<StackPanel>
    <TextBox x:Name="Token" />
    <Button x:Name="Login" Content="Login to System" CommandParameter="{Binding ElementName=Token, Path=Text}" />
</StackPanel>
```

If you're looking to combine the best of [Caliburn.Micro][cm] and [ReactiveUI][rxui] then this may help.

We can then

[rxui]: https://reactiveui.net/
[ndc]: http://ndcsydney.com/
[cm]: http://caliburnmicro.com/
