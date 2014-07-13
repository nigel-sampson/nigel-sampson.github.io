---
layout: post
title: Back in business
tags: 
---

Wow, that took a lot longer than expected. Vodafone took every single
day of their &quot;up to three weeks for your broadband to be connected&quot;. In
the last few weeks I&#39;ve taken up my new job and am settling in fairly
well. Part of the new job entails evaluating new technologies (more so
than I usual do) for use within the company. After reading over the
Entity Framework I have mixed feelings.Visual designers are always
pretty much as many have people have pointed out they scale terribly,
the strongly typed querying via LINQ, but hopefully that can come to
libraries such as NHibernate soon. The lack of implicit lazy load is
also really annoying. 

In my opinion the EF designer should
only map the relationship between the conceptual and the data models.
Not generate the conceptual model, this tightly couples them and
designer generated code is always a pain in the ass to extend (partial
classes only alleviate some of this pain). The post [DP Advisory Council](ttp://blogs.msdn.com/dsimmons/archive/2008/06/03/dp-advisory-council.aspx) gives me a lot of hope in this regard.

Over the week or so I&#39;ll outline the project I&#39;d like start to build in my
spare time and hopefully share some of it with you in the same vein as [MVC Storefront](http://blog.wekeroad.com/mvc-storefront/mvc-storefront-part-1/) by Rob Connery. Probably not via screencasts, but a product built from the ground up.


