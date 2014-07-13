---
layout: post
title: Using Blog Engine .NET
tags: 
---

So I decided to use Blog Engine .NET for the foundation of this blog, I
suspect I&#39;ll move to something homegrown when I start putting together
a product oriented site, but for now it certainly scatches the itch.

So what did I think of it?

It&#39;s an excellent blogging solution out of the box, the only things I
did that I disagreed with were mostly matters of opinion in such things
as urls and I could have changed this since it&#39;s open source.

It&#39;s only when I wanted to create my own theme that I started to run
into problems. For a number of the pages the html generation was all
over the place, a crazy mixture of strings in the code behind,
generation of html dom (new HtmlTable() etc) and in the aspx file
itself. Overall this slowed me down in altering the look and feel, and
for some pages I gave up entirely. The ajax on the contact us and
comments pages seem very fragile as well.

In conclusion, for out of the box functionality it&#39;s great, for
altering the look and feel it&#39;ll require quite a bit of untangling and
time investment to get it right.

At some point I&#39;ll invest time into MVC and Silverlight as a learning
exercise and see what I can do myself (all programmers think they can
do everything better), but for me it&#39;s balancing &quot;Not Invented Here&quot;
against having a real world example to learn a new set of technologies.
