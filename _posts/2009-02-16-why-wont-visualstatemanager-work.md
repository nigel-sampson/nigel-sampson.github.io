---
layout: post
title: Why won't VisualStateManager work?
tags: silverlight
---

As someone who&#39;s still learning Silverlight I like to put together the
xaml for all my examples and experiments by hand. Since xaml and the
underlying object structure for a control are exactly the same once you
can compose something in xaml you can do the same in C#. 

One problem with hand composing all the xaml is that I&#39;m not as smart
as software like Blend and often make mistakes. Most of the time the
exception details make it pretty easy to diagnose where I&#39;m going
wrong. However while writing the previous post about using the visual
state manager to style the focus panel I encountered a nasty looking
exception when I call VisualStateManager.GoToState. 

Error: Unhandled Error in Silverlight 2 Application Catastrophic failure (Exception from HRESULT: 0x8000FFFF (E_UNEXPECTED)) at System.Windows.VisualState.EnsureStoryboards()

The
rest of the stack trace isn&#39;t important suffice to say the internal
exception details proved no help. The problem is that the storyboard
for that state is suffering a xaml parse issue but doesn&#39;t tell you
what.

So how to diagnose? After half an hour of trying random things (bad
practise I know) with no luck I found a much easier way to locate my
issue. I simply created a new user control, extracted the style
template and the storyboard. Bingo! 5 seconds later a nice exception
telling me I had missed a &quot;#&quot; on one of the colour codes.

So much easier! Hope this helps someone.


