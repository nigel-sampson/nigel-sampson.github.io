---
layout: post
title: To var or not to var
tags: csharp
---

Lately I&#39;ve been seeing a lot of blog examples using the var keyword
from C# 3.5 in places where the type name is well known. A good example
of this would be at &quot;[Comparing Moq to Rhino Mocks](http://haacked.com/archive/2008/03/23/comparing-moq-to-rhino-mocks.aspx)&quot;
by Phil Haack.&nbsp; In my opinion it makes the code less readable, possibly
not from the person in the IDE, but has someone reading a code sample
on a blog it removes some of the explicitness around the types being
used.

Personally I&#39;ll be using the var keyword only when I need to i.e: when using anonymous types.

**Edit from 2010**: I was wrong, var is awesome :-)

