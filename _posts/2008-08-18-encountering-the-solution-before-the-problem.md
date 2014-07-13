---
layout: post
title: Encountering the Solution before the Problem
tags: 
---

I read a post a few weeks ago about feeling the pain of some software
development practises including such things as untestable code and
tight coupling. I&#39;d really like to post a link to it however Google
Reader search isn&#39;t at all helping in helping me re find it. But first
some history.

I feel like I&#39;m in that
position now, during my previous employment I encountered during my
R&amp;D a lot of what&#39;s now labelled as ALT.NET practises such as
Dependency Injection containers, NHibernate, TDD andDDD . These
practises really resonated with me and since I was leading the
development of a new product I was a in good position to get to grips
with some of them (I still have trouble with test first development)
with mixed results (another post).

Coming
back to NZ and slotting into my old position I started with a lot of
enthusiasm for bringing my team &quot;into the light&quot; and sharing what I
thought was a better way to build applications. Our company is
marketing services based so the web sites we build can vary from the
small brochure sites up to fully integrated e-commerce applications
(very very bespoke). In hallway discussions with other programmers I
talked about some of what I had learnt and encountered mixed reactions.

The
overall impressions I received were that since our team had not
encountered some of the pain that these patterns solve then their
perceived value was lessened. I also feel some of the pain our team has
felt could be caused by these but attributed elsewhere (the functional
spec is always a tempting scapegoat).

Our
client&#39;s domains do not have huge dramatic shifts (I can only think of
one counter example) so the pain of tight coupling isn&#39;t as strong.
Without having to develop different implementations of services for
unit testing or having the domain implement core service interfaces
lessens the pain of not having dependency injection.

While
I still believed that these practises could helpful for us to use I
needed to think differently about they should be approached and the
seminal question. *What to do when you encounter the solution before experiencing the pain of the problem?*

In practise we do this all the time, we build our
solutions on top of other solutions, we use frameworks, libraries and
patterns, all proven solution for time old problems, we certainly don&#39;t
reinvent the wheel every time.

I&#39;m finding
now there needs to be a bit of mix, the pain needs to be there, but
only in small doses. Create the problem yourself, show a bit of pain,
and then &quot;solve it&quot;.

A good example is
right now I&#39;m building some simple content management capabilities into
our core framework. Part of which is some routing based off the new MVC
Routing engine (another post). Due to the pages (and the page
repository) being part of the domain my core services are programmed
against interfaces (IPage and IPageRepository). Obviously these need use so we have a simple &quot;RepositoryLocator&quot;
as this pattern of usage grows and starts to become unwieldy I&#39;ll
introduce a container such as Windsor or Unity to reduce the pain.

In
short, I think some developers really need to encounter a problem
before the value of the solution is perceived, but if we can do this in
a controlled fashion then everyone wins.                

