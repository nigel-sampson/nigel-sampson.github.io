---
layout: post
title: Issues with ListPicker
tags: windows-phone
---

Just a bit of a heads up to people, I had an application update to [To Do Today]/windows-phone/to-do) fail validation for the Marketplace, due to an issue with ListPicker control in the [Silverlight for Windows Phone Toolkit](http://silverlight.codeplex.com/).

Specifically the requirement failed was *"If the current page displays a context menu or a
dialog, the pressing of the Back button must close the menu or dialog and cancel the backward navigation to the previous page."*

I was a little dubious to start with does the smaller ListPicker count as a menu? Checking the Calendar application it looks like the back button does in fact close an expanded ListPicker.

The Silverlight Toolkit team are on the ball with a [work item loaded into Codeplex](http://silverlight.codeplex.com/workitem/7643) so I highly recommend going and updating any references you have.
