---
layout: post
title: Caliburn&#58; Binding Conventions
tags: csharp silverlight wpf caliburn
---

Lately I've been playing around with Caliburn in WPF (and a little in Silverlight). One of the things I've really enjoyed is the convention based binding between the View and the ViewModel (Presenter). The default binding (DefaultBinder) helps with the more complex screen activiation by binding the Presenters collection and exposed Presenter objects on the ViewModel. It does all this binding based on the name of the control and the exposed property.

One thing it doesn't do is bind any simple exposed properties to the view, such has simple strings, collections and so forth. To implement this we'll create a new IBinder, to ensure we keep all the existing conventions we'll inherit our new ConventionBinder from DefaultBinder and override ApplyBindingConventions.

The logic for this will be fairly simple, we'll loop through all public instance variables on the model and look for a control of the same name. If we find one we'll look up which property to bind to and ensure the types are comparable, after that we create the Binding object based on the type of property we're binding to.

``` csharp
public class ConventionBinder : DefaultBinder
{
    private readonly Dictionary<Type, DependencyProperty> defaultProperties = new Dictionary<Type, DependencyProperty>
    {
        {typeof(TextBox), TextBox.TextProperty},
        {typeof(TextBlock), TextBlock.TextProperty},
        {typeof(ItemsControl), ItemsControl.ItemsSourceProperty}
    };
 
    public ConventionBinder(IServiceLocator serviceLocator, IActionFactory actionFactory, IMessageBinder messageBinder)
        : base(serviceLocator, actionFactory, messageBinder)
    {
    }
 
    protected override void ApplyBindingConventions(DependencyObject element, object model)
    {
        base.ApplyBindingConventions(element, model);
 
        if(!(element is FrameworkElement))
            return;
 
        foreach(var property in model.GetType().GetProperties(BindingFlags.Public | BindingFlags.Instance))
        {
            var bindingElement = FindControl(element, property.Name);
 
            if(bindingElement == null)
                continue;
 
            var defaultProperty = GetDefaultProperty(bindingElement.GetType());
 
            if(defaultProperty == null)
                continue;
 
            if(!defaultProperty.PropertyType.IsAssignableFrom(property.PropertyType))
                continue;
 
            BindingOperations.SetBinding(bindingElement, defaultProperty, new Binding(property.Name)
            {
                Mode = GetDirectionForProperty(property)
            });
        }
    }
 
    private DependencyProperty GetDefaultProperty(Type elementType)
    {
        do
        {
            if(defaultProperties.ContainsKey(elementType))
                return defaultProperties[elementType];
 
            elementType = elementType.BaseType;
 
        } while(elementType != null);
 
        return null;
    }
 
    protected virtual BindingMode GetDirectionForProperty(PropertyInfo property)
    {
        if(property.CanRead && property.CanWrite)
            return BindingMode.TwoWay;
 
        if(property.CanRead)
            return BindingMode.OneWay;
 
        if(property.CanWrite)
            return BindingMode.OneWayToSource;
 
        return BindingMode.Default;
    }
}
```

Now we just need to configure Caliburn to use the new binder. Now all properties will be bound to the controls with the appropriate name.

``` csharp
protected override void ConfigurePresentationFramework(PresentationFrameworkModule module)
{
    module.UsingBinder<ConventionBinder>();
}
```

The list of default properties is in no way exhaustive, just add to it when you want a new convention.
