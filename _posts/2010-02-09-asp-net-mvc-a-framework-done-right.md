---
layout: post
title: ASP.NET MVC&#58; A Framework done right
tags: mvc silverlight wpf
---

Lately I've been doing a lot of miscellaneous work in ASP.NET MVC, Silverlight and WPF and some of the differences in approach by their respective development teams is infuriating. The MVC libraries have some of the best examples of how to build a solid library while still allowing developers to override granular pieces of behavior. The amount of extensiblity [currently available](http://codeclimber.net.nz/archive/2009/04/08/13-asp.net-mvc-extensibility-points-you-have-to-know.aspx) is amazing (with more coming for validation in the next release).

Compared to this WPF has way too much stuff marked as internal, private or sealed, a good example of how this inhibits some great innovation is "[Forget about extending WPF data binding support](http://www.clariusconsulting.net/blogs/kzu/archive/2007/09/23/ForgetaboutextendingWPFdatabindingsupport.aspx)".

I'm not a huge fan of the IDataErrorInfo interface, it being a kind of hold over from .NET 1.0 DataSet days, so wanted to create similar interface to work with a validation system seperate from the model. WPF has built in support for IDataErrorInfo through the DataErrorValidationRule and my initial approach was to build something similar for my own interface.

Sadly 90% of the code required for DataErrorValidationRule depends on the internals of BindingExpression so no dice. I'm finding it very frustating that something as simple as DataErrorValidationRule couldn't be implemented by a developer outside of Microsoft, what hope do the rest of us have at making meaningful extensions to the way WPF / Silverlight works?

The best example I've found at doing this is "[Automatically validating business entities in WPF using custom binding and attributes](http://www.codeproject.com/KB/WPF/wpf_custom_validation.aspx)" which requires a good chunk of wiring code thanks to the [BindingDecoratorBase](http://www.hardcodet.net/2008/04/wpf-custom-binding-class) class.</p>

A bit of rant I'm afraid, but had to get some frustration at the lack of extensiblty in WPF / Silverlight validation.
