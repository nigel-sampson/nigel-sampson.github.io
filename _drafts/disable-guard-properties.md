---
layout: post
title: How to stop guard properties disabling your controls
tags: csharp xamarin caliburn-micro
---

Caliburn.Micro has a really useful feature in **guard properties**, these are in essence a predicate property that is paired with a method and returns true or false depending on whether the method should be called. If you're used to working command objects then the guard properties are akin 

``` csharp
ActionMessage.ApplyAvailabilityEffect = (context) =>
{
	if (context.Source is DataGrid)
		return true;

	return applyAvailabilityEffect(context);
};
```