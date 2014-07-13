---
layout: post
title: Is being Blendable worth it?
tags: silverlight
---

So I've been going over this with a few others lately, is ensuring your project is "Blendable" worth it?

The Model, View, ViewModel pattern does a great job in separating the UI from the logic for each screen, at some point however they have to be integrated. This has been the part I've been showing in the screencasts of this series. Going over this I feel there are a few areas that ensure good usage of Expression Blend.

* Being able to create and edit a View using Blend, ultimately this means a parameter-less constructor no business logic in the constructor (shouldn't be doing this with MVVM anyway).
* Being able to integrate the View and ViewModel, originally I tried to use the Blend UI as much as possible for this, but ultimately I don't think it's feasible, a lot of the integration will be done in xaml. The Blend UI doesn't have enough features to be able to do all the integration, it could be possible with a plethora of Behaviors for each custom task but it could get complicated.

Overall while Blend is a great tool for putting together your view it doesn't do well for integrating the view and view model. I think it's still useful to ensure Blend is usable, but limiting yourself to only what you can do in the Blend UI is counter productive.
