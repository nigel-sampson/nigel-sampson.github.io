---
layout: post
title: Auto subscription for Event Aggregator
tags: csharp xamarin caliburn-micro
---

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