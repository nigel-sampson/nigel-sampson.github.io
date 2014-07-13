---
layout: post
title: NDepend&#58; A Review
tags: 
---

So a few weeks I was sent a license to [NDepend](http://www.ndepend.com/) by Patrick Smacchia and asked if I'd discuss it if I found it useful. NDepend is an analysis tool that tries to help in understanding the metrics of .NET assemblies, one very interesting feature is the ability to query yourcodebase using CQL (Code Query Language).

Being the typical geek I fired up it and pointed at an old codebase, once it was done I was blown away by the amount of information available to me, maybe a little too much. By default the UI feels very busy but I suspect that may lessen once I grow used to using it. The [getting started demos](http://www.ndepend.com/GettingStarted.aspx) helped a huge amount at explaining what I was looking at.

By default there are also a huge selection of built in CQL queries that can provide warnings and errors during the analysis process, for instance warning about classes with too many methods (**WARN IF Count &gt; 0 IN SELECT TOP 10 TYPES WHERENbMethods &gt; 20 ORDER BY NbMethods DESC** ). Being able to define a teams coding principles into solid rules that can be integrated into a CI process would really be interesting (NDepend comes with MSBuild and Nant tasks).

Overall my initial impressions are favourable, it feels a lot like Resharper in that initially it can feel like too much is in your face and can force people away, but I feel that like Resharper this could be a very useful tool in the long run.
