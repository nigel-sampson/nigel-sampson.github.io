---
layout: post
title: The nature of breaking changes
tags: csharp caliburn-micro
---

[Caliburn.Micro][cm] like most .NET open source projects these days tries to follow [Semantic Versioning][sv], it helps with Nuget and [versioning dependencies][ng] and gives some measure of confidence to users of libraries updating libraries to newer versions.

One of the major guidelines is: 

> Increment the MAJOR version when you make incompatible API changes.

which in my opinion is a very good approach and one we should strive to achieve. But what is "an incompatible API change" or as it's otherwise known "a breaking change"?

We're looking at bringing a decent chunk of major functionality in [Caliburn.Micro][cm] and want to do it in way that doesn't elicit us pushing to v10 in a few months.

There's no hard and fast answer, if a user of your assembly depended on a bug then fixing that bug would be a breaking change for them.

I remember reading an article from the .NET team where they had to consider changing the signature of private methods as a potential breaking change due to the use of reflection. I'd consider that overkill for almost all developers. However adding extra overloads or adding optional parameters to public methods could break code that depended on reflection.

Sometimes breaking changes can because by the interactions of different libraries, a great example from [Caliburn.Micro][cm] was where we added `[DataContact]` to `PropertyChangedBase`, a fairly innocent change to allow people to serialize their view models with the out of the box serializers. What we didn't know was that [Json.NET][jn] respects that property and this was a breaking change for anyone using that serializer. 

I don't think you're ever going to find a hard and fast rule about what consists a breaking change. Each change needs to be evaluated on a case by case basis and thoughts on how developer are most commonly using your libraries.

What are your thoughts?

[cm]: http://caliburnmicro.com/
[sv]: http://semver.org/
[ng]: http://docs.nuget.org/docs/reference/versioning
[jn]: http://james.newtonking.com/json