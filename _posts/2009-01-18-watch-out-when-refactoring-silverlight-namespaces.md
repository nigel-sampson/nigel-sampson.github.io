---
layout: post
title: Watch out when refactoring Silverlight namespaces
tags: silverlight
---

This was a small problem that had me stumped for around 10 minutes. Often I have the default namespace and assembly (and .xap) name to a project differ from the project name. For example if the solution is **CompiledExperience.Examples** and the project is *Animation.Position* then the namespace I&#39;ll set as *CompiledExperience.Examples.Animation.Position *for consistencies sake.

However
the first time I did this in a Silverlight project things broke.
Everything compiled fine however after the standard Silverlight loading
screen .... nothing. Turns out just like Console applications in .NET
Silverlight has the concept of a start up object. In Console projects
it&#39;s the class that contains the Main method you&#39;d like to use. In
Silverlight it&#39;s the one that inherits from Application that you want
to use (as ultimately this class will handle initial visuals and error
handling). 

You can set this start up Application object in the
Silverlight tab of the project properties. My problem was that because
I had moved this object into a different namespace then nothing
happened after startup. Simply setting the object again in the project
properties had everything going again smoothly.
