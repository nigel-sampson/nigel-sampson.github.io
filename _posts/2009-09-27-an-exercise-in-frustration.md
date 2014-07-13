---
layout: post
title: An exercise in frustration
tags: silverlight
---

This is a bit of rant to get some things off my chest about Silverlight application development. While building the demo application for last weeks post I basically had to throw away the initial concept due to not being able to get it with with a level of code I deemed acceptable for an introductory post.

My original intention was to have an application where the user selected what ingredients they had and the system returned the cocktails that could be made. From theViewModel point of view I wanted to expose two collections (Ingredients and Cocktails) and a ICommand that took the selected ingredients as a parameter. This should be pretty simple, but it was nothing of the sort, here's what I encountered.

*Behaviors not bindable:* I've talked about this before and it's still an frustration for me, you can get around most of it with the code from Pete Blois, but the ExecuteCommandAction is nowhere near as simple as it should be. Sadly until the binding issue is resolved we have to use it (or we could use a non blendable custom attached property).

*ListBox.SelectedItems:* Turns out this property doesn't support binding either, this makes having a simple ElementName binding from the CommandParameter impossible. I think it should be bindable and non read only. There are solutions out there involving creating your bindable lists using custom attached properties, I had a crack at using a Blend behavior to do the same thing, but turns out Behaviors can't be targets ofElementName binding (more on this in a second).

I decided to fall back to code behind to wire together the selected items to the command parameter, then found that giving an action a x:Name doesn't make it accessible from the code-behind (internally this field is hooked up with FindName) which doesn't locate actions.

In my mind this should have been a very easy example to show off the functionality of Silverlight 3 (multi select list boxes, interactivity with behaviors and element name binding). But of it worked in a expected way, I probably could have cobbled together a working prototype but the amount of **"this doesn't work correctly so here's a work around"** makes it useless as an introductory post into MVVM.

So where to from here? I'd love to see these issues resolved (from the Silverlight 4 wishlist I can see that full data binding is a highly requested feature). I can see that improving things vastly, the amount of ceremony you need due to behaviors not beingbindable (and not immediately accessible from code behind) mean there's way too much plumbing.

Sorry for the rant, but there was a lot of frustration that I needed to clear up.
