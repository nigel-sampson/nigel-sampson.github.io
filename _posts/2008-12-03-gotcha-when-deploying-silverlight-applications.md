---
layout: post
title: Gotcha when deploying Silverlight applications
tags: silverlight
---

I&#39;m just in the process of writing my first Silverlight post, one of
the benefits of writing about Silverlight is that you can show it in
action. So last night I put together the working example, and was
running into a roadblock at the final stage, deployment. 

I was using the Silverlight control in the System.Web.Silverlight.dll
assembly and no matter what I did with IIS mime types and minimum
versions it would always come up with the &quot;Get Silverlight image&quot;. It
turns out that I was too quick to get the latest bits for Silverlight 2
when it was finally released, the final version of the Silverlight
Tools for Visual Studio (and therefore the aforementioned assembly)
weren&#39;t released till approximately a week later.

The reason I noticed it was the type attribute on the &lt;object&gt;
output, it mentioned beta 2 and not the standard
&quot;application/x-silverlight-2&quot;.

Little frustrating but glad it&#39;s solved.

