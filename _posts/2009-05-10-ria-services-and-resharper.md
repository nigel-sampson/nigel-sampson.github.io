---
layout: post
title: RIA Services and Resharper
tags:  csharp silverlight
---

Wow that took a seriously long time, Vodafone customer service was the worst I've experienced in many years. This was a little gotcha I experienced while at "[Expression For Arts Sake](http://timheuer.com/blog/archive/2009/04/30/expression-for-arts-sake-wellington.aspx)" and took me a bit to work out what was going wrong.

.NET Ria Services is an interesting piece of technology, in my mind it's the love child of ADO.NET Data Services and Dynamic Data. The ability to expose your model over the wire to a Rich Internet Application including data annotations to assist in scaffolding a working UI on the other end. I haven't had too much of a play yet, I'm hoping it's as extensible as Dynamic Data and MVC such that I can plug in different ways to compose that model (not just attributes on Metadata classes).

Given that the point of EFAS was to play with new technology I thought I'd try RIA Services as the technology to pull data down from my website and database. Setting it was pretty quick, but for the life of me I couldn't get it to work. Visual Studio was complaining about missing classes and namespaces. After about an hour of pulling my hair out and trying the simplest scenarios it still wouldn't recognise the generated DomainContext and classes. This is what Visual Studio looked like.

![Resharper errors](/content/images/posts/resharper.png)

And then I just hit build and it did! After recovering from shock I realised what had happened. Turning Resharper off suddenly Visual Studio looked like this.

![No Resharper errors](/content/images/posts/visual-studio.png)

The reason is that the client side code for RIA Services isn't "technically" part of the Silverlight project. It's just in a folder named "Generated Code" that Visual Studio includes as part of it's intellisense and build steps. This means that add ins like Resharper don't believe it's part of the project and throw up errors as before.

I'd really like to see RIA Services follow the more traditional Service Reference pattern with an option to "Update Service Reference on project build" to get around shared code and so on. This was the generated proxy is part of the project and avoids yet another way to have generated code.

Hope this saves someone some frustration.
