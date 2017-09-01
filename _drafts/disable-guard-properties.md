---
layout: post
title: How to stop guard properties diabling your controls
tags: csharp xamarin caliburn-micro
---

``` csharp
ActionMessage.ApplyAvailabilityEffect = (context) =>
{
	if (context.Source is DataGrid)
		return true;

	return applyAvailabilityEffect(context);
};
```