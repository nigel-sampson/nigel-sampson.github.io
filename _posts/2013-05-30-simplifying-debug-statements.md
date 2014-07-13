---
layout: post
title: Simplifying debug statements
tags: csharp
---

Often I find myself using Debug statements in order to test code where breakpoints wouldn't work. Usually where testing how numbers changes over time due to user interaction such as scrolling or camera input. Debug.Writeline does thankfully follow the same structure as Console.Writeline which means I usually end up creating a line something along the lines of.

``` csharp
Debug.WriteLine("X: {0}, Y: {1}, Horizontal Offset {2}", x, y, horizontalOffset);
```

Recently I discovered a couple of little compiler tricks that make it easier to get debug output. The first is the C# compiler creates a ToString implementation for all anonymous types. The second thanks to Resharper is that when declaring an anonymous type if the property name is the same as the variable you don't need to declare the property name.

Combining the two tricks lets me rewrite the line as follows.

``` csharp
Debug.WriteLine(new { x, y, horizontalOffset });
```

Which gives us output that looks like.

```
{ x = 45.23675, y = -13.3658, horizontalOffset = 63.4586 }
```
