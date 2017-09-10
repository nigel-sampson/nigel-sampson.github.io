---
layout: post
title: Auto subscription for Event Aggregator
tags: csharp xamarin caliburn-micro
---

In many of my talks I've recommended using a messenger style class. These help to reduce coupling between view models when decomposing from a single "view model per screen" to a tree of view models. This intermediary class is called by a few different names, `Messenger` in Xamarin.Forms, `Mediator` in some others and in Caliburn.Micro, `Event Aggregator`.

One feature of the `Event Aggregator` in Caliburn.Micro is that you need to explicitly subscribe to it through `IEventAggregator.Subscribe` before you'll receive events from it. This is by design to be able to integrate the aggreator with the life cycle of your view models. Typically most view models will only want to receive events while they've active, some however want to receive them all the time.

Some developers would prefer an "auto subscription" style behavior where view models are automatically subscribed to the aggregator when they're used. One benefit of this approach is that the view model doesn't need to take a dependency on aggregator itself reducing some complexity.

The best cut point to introduce this behavior in Caliburn.Micro is the `ViewModelBinder.Bind`, this is the part of the framework that once a view and view model are located they're "bound" together. This `ViewModelBinder` is used by the `INavigationService` and the `View.Model` attached property and covers a few things:

1. Sets the `DataContext / BindingContext` of the view to the view model.
2. Applies the property conventions (not for Xamarin.Forms).
3. Applies the method conventions (not for Xamarin.Forms).

A lot of the extension points in Caliburn.Micro like this one are defined as static properties, in this case of type `Action<object, DependencyObject, object>`. The way we modify it is by setting it to a new action that includes the new behavior, we can preserve the existing functionality by taking a reference to the current action and calling it within the new action. This looks like the following:

``` csharp
var existingBind = ViewModelBinder.Bind;

ViewModelBinder.Bind = (viewModel, view, context) =>
{
	existingBind(viewModel, view, context);

	var handleInterfaces = new[]
	{
		typeof(IHandle<>),
		typeof(IHandleWithCoroutine<>),
		typeof(IHandleWithTask<>)
	};

	var subscribe = viewModel
		.GetType()
		.GetInterfaces()
		.Any(i => i.IsGenericType && handleInterfaces.Contains(i.GetGenericTypeDefinition()));

	if (!subscribe)
		return;

	var eventAggregator = container.GetInstance<IEventAggregator>();

	eventAggregator.Subscribe(viewModel);

	var deactivate = viewModel as IDeactivate;

	if (deactivate != null)
	{
		deactivate.Deactivated += (s, e) =>
		{
			if (e.WasClosed)
				eventAggregator.Unsubscribe(this);
		};
	}
};
```

In the above code we're checking whether the view model implements one of `IHandle*` interfaces that's required by the event aggregator, if it doesn't we can exit out otherwise we'll subscribe the view model to the event aggregator. We'll then check if the view model supports deactivation, if it does we'll attach to the `Deactivated` event and if we're closing the view model we'll unsubscribe.

Unsubscribing is important because although the event aggregator itself holds a weak reference to your view model, you have the possibility of discarded but not yet garbage collected view models receiving events and acting erroneously.

This post should show how we can extend Caliburn.Micro to add automatic behavior to view models without too much extra complexity.