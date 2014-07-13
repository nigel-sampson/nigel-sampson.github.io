---
layout: post
title: Compiled Experience Relaunched
tags: 
---

As a few eagle eyed readers may notice the Compiled Experience has been redeveloped, I've migrated away from BlogEngine.NET to a homegrown solution on top of Asp.Net MVC, NHibernate and Castle Windsor

The reason behind doing this was two fold, I wanted to make some major additions to the website and I felt more comfortable building on top of something I had written myself, the other reason was wanting to learn MVC. I find it easier to stick to a learning project if there's a valid reason behind it.

There are some teething problems, mostly around legacy urls which hopefully will be addressed tonight so please bear with me.

### My Thoughts on MVC

Developing using the Asp.Net MVC platform was certainly interesting, I certainly agree with a lot of the comments around that MVC gives the tools to create your own MVC Framework by extending it with the conventions you want (It allows you to build the gun to shoot yourself in the foot with). Very quickly do you realise how much Web Forms abstracts you from HTTP and HTML as things like validation, sitemaps and ajax do not come easily out of the box.

Even with all this I really enjoyed the feeling of absolute control over the website while building this website. Not feeling a little ill seeing the generated HTML was good, the largest hold ups for me was certainly the View of MVC. I'm not a fan of the large amount of string concatenation that comes along with HtmlHelpers, and trying to simplify the code used in the View (no big decisions etc) was difficult in parts.

I'm glad to see RenderAction making it into MVC vNext as I feel this is the best way to render parts of the view that are orthogonal to the view itself. The biggest hassle I had in the end was coming up with a validation system I was happy with. Combining client side validation, standard server validation as well as model state errors being sent down via a JSON request required a little tweaking of jquery.validate but I got there in the end.

### My Thoughts on Development in general

I think as developers we have a tendency to wait until we believe our development is perfect and then unleash it on the world with trumpets and thunderous applause (hopefully). This is bad in my opinion, I like the idea of "release early, release often" and "if you're not embarrassed by your first release then you waited too long". I certainly left out a chunk of functionality I wanted to include and plan to do more regular updates to the website (every couple of weeks).

Unit testing is something I always find interesting, in long running applications having unit tests makes maintenance a lot less of a hassle. This is where you get the real value of unit tests, in the long run. I could write many many paragraphs on the value of these tests, but I still sometimes find it hard to keep writing them, often I found myself in a "productivity binge" around writing lots and lots of new untested code and having to go back to write tests.

Overall I hope you enjoy the new site, I'll post when there are some new interesting features, let me know if something is breaking or if you have any comments in general.
