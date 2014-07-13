---
layout: post
title: Don't return Action Result (if you can help it)
tags: mvc csharp
---

A while ago I was looking into dynamic languages (in particular Iron Python) and an idea presented in a talk stuck with me. I wish I could remember who it was to credit them. In essence it was that in dynamic languages unit tests verify what the compiler can't enforce and that we can think of the compiler as simply another unit test in the system. This was obviously framed in a way that meant that the type safety wasn't necessary but lets approach it from a different direction.

### Don't unit test what you can use the compiler to enforce.

A common thing I see in MVC examples is a unit testing verifying that some controller action returned the correct type of ActionResult e.g **IndexReturnsViewResult**. While this test is useful, we can remove it by changing the return type of Index to ViewResult, now the compiler enforces our test for us! This clearly won't work for actions that return different types of result depending on context (this is where unit testing should be testing). But if we can have our compiler enforce some of our specification then I think we should.
