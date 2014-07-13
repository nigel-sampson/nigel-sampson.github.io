---
layout: post
title: Integrating Sandcastle into your builds
tags: msbuild
---

So the opportunity to write this post came up a little quicker than anticipated.

Recently
I&#39;ve been tinkering with using MSBuild to create full build scripts
with the new view of introducing my coworkers to concepts such as [Continuous Integration](http://martinfowler.com/articles/continuousIntegration.html),
one of these steps is building the documentation for our core
libraries. In the olden days of .NET 1.1 I used NDoc for this but sadly
it&#39;s no more so we&#39;ll be using [Sandcastle](http://blogs.msdn.com/sandcastle/).
Now a quick Google search will let you download some MSBuild targets
for Sandcastle, however I had no luck with them, I suspect this was
because of new versions of the Sandcastle and/or incorrect piping
between the different tools in Sandcastle.

So what&#39;s the easier way?

[Sandcastle Help File Builder](http://www.codeplex.com/SHFB)
is an excellent tool akin to NDoc that allows you to graphically set up
and build your Sandcastle documentation, it&#39;s also a good way to
quickly verify how you want your documentation to look before
introducing it to your build script.

The best part? As well as
having a GUI it also comes with a Console version, and by saving your
documentation preferences into one of their project files (*.shfb) it
can also easily be maintained in version control. The build script
Target looks something like this=

``` xml
<Target Name="Documentation">
  <Exec Command="$(SandCastleHelpBuilderPath) Infrastructure.Core.shfb" />
</Target>
```

Hope this helps someone.

