---
layout: post
title: Thoughts on the MIX 11 announcements
tags: windows-phone
---

There's been a lot of awesome announcements coming from Microsoft about the Windows Phone 7 at [Mix](http://live.visitmix.com/) this year. I really wish I had the chance to be there in person but sadly with a fledgling business in Compiled Experience tickets from Auckland to Las Vegas are a bit too much. Maybe next year.

It really looks like they've taken a lot of developer feedback to heart, especially in the areas of integration between the phone and the app. It's an area that was very locked down in the first version and is gradually being opened up. This seems to be a theme in Silverlight as well, hopefully it continues for future versions.

I'm especially happy to see the live tile and background agent support, it's a feature users have been screaming out for in [To Do Today](http://compiledexperience.com/windows-phone-7/to-do) that couldn't be built to my satisfaction using the current methods. It's the first thing I'll be doing as soon as Mango is released. The deep linking functionality of live tiles and toast notifications will certainly be interesting. It's certainly going to make in app navigation a lot more complex given that you won't entirely be sure of the navigation stack any more as you'll need to add home buttons.

One thing that wasn't really announced but only briefly mentioned (as part of the KIK demo if I remember correctly) is that Mango will include Silverlight 4. This is a great thing for me if it includes Binding to non Framework Elements (always a bugbear of mine for Silverlight in the browser). This means binding to things like behaviours without hacks. Hopefully it also adds some support for validation as there's pretty much nothing really supported in WP7 as it stands.

Better binding really adds value to users of the [Caliburn Micro](http://caliburnmicro.codeplex.com/) framework, one awesome feature is convention based binding to parameters of actions which currently isn't possible due to the binding infrastructure. By having the binding on parameters it makes the guard actions more useful as they're refreshed whenever a dependent control changes. It'll make view models feel more natural rather than property bags with a lot of parameter-less actions.

SQL CE on the phone will be helpful, it'll be interesting to see the performance of it compared to serializing documents to Isolated Storage. I'd love to see an embedded document database ala Raven or Mongo.

Lately I've been using the [EQATEC profiler](http://www.eqatec.com/Profiler/Overview.aspx) but one that's built into Visual Studio with real hooks into the visual tree and storyboards will be very useful. And best of all these tools look to be available next month! There won't be a need to build in fake accelerometers and GPS services any more and it will be nice to profile my existing apps.

A number of things are missing for me from the announcements, the first was anything around design resources. It really appears that a lot of developers are equating Metro to not needing to put some thought into design and layout. There's a huge amount of "fugly" applications out there that have really put me off purchasing them. What's missing from a lot of applications is good animation and movement, it's a huge part of Metro and not having a standard library of animations is really hurting user built applications.

The second thing was anything around the marketplace for developers, better break downs of downloads versus sales, being able to export the data to other formats. Revenue reports and things like time between download of the trial and sale. Some of this can be mitigated by using your own analytics solution, but if it's baked it then it'd certainly be better. Also being able to submit an update to a patch and to have to repopulate all the details of the app.

Microsoft really will need to keep on top of the update process, if the same issues with NoDo come up again with Mango there will be hell to pay.
