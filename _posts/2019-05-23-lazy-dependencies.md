---
layout: post
title: Be careful of lazy dependencies
tags: csharp graphql
---

## What are lazy dependencies?

.NET Core has a lot more built in support for dependency injection which is an incredibly useful feature and this post isn't about not using that, but to think about how and when your depdendencies are created. By dependencies I mean any object being created by your Container which is a pretty broad definition which typically covers controllers, services and more.

My definition of a lazy dependency is on that's created by the container the first time it's required. I think of it as being orthognal to the lifetime of the dependency as singleton dependecies can be lazy as well as transient ones (the differences is in how often they're created and not when). For most our services this isn't an issue as often the creation of our dependency doesn't execute any significant code, the constructors often look something like the following:

``` csharp
public ProductsController(IProductRepository repository, IEventBus events)
{
    this.repository = repository;
    this.events = events;
}
```

This code is unlikely to fail so I don't have concerns about `ProductsController` being lazy. In fact most of our dependencies are lazy (the one's that aren't are usually the ones registered as already instantiated objects into the container) and it's not a problem.

## Which ones should I be careful of?

If a majority of our dependencies are lazy what are the ones I should be careful of? In my opinion it's the ones that have any significant behaviour in their constructors, this could be reading files or opening connections for example. If our `IEventBus` implementation establishes a connection to the messaging solution on instantiation then we need to think about what happens if it fails or takes a significant amount of time.

This lazy evaluation means the first request to the `ProductsController` will be slower due to paying the cost of connecting to the message bus or may fail if that connection fails.

A more concrete example comes from our work implementing a GraphQL server using [Hot Chocolate][hc]. In GraphQL frameworks you start with the definition of your GraphQL schema, in Hot Chocolate this is represented by the `ISchema` interface and configured at start up. However the actual creation of the schema is lazy and doesn't happy until the first GraphQL request is received which is a little slower but if I've misconfigured my schema I don't see the error till that first request instead of the application startup.

## Why failing fast can be good

Modern deployment strategies (with and without containers) tend to involve standing up the new verison of the application, moving traffic across to the new version and then tearing down the previous version. This enables a seamless deployment experience for the users of the application. However for this to happen the system needs to be able to determine if the system is healthy and the old version can be torn down. If I create a new version where the schema is misconfigured then what want to see happen is the new version is deployed but at start up fails fast, this can then short circuit the deployment system such that previous version keeps serving traffic. 

Other examples of things we fail fast on are if we detect that there are pending database migrations, this signals that we're deploying code that is depending on a database migration that hasn't been deployed to the database. We never want that code to execute so we fail fast and leave the previous version executing.

## Forcing evaluation

In the examples above we can fail fast by throwing exceptions (this is what we do when there are pending migrations), but for our GraphQL schema example we have a simple extension method.

``` csharp
/// <summary>
/// Resolves the GraphQL schema forcing validation and ensures any errors before start up failures
/// </summary>
public static IApplicationBuilder ResolveGraphQLSchema(this IApplicationBuilder applicationBuilder)
{
	try {
		applicationBuilder.ApplicationServices.GetRequiredService<ISchema>();

		return applicationBuilder;
	} catch (Exception ex) {
		throw new StartupFailureException("Failed to resolve the sitched GraphQL schema", ex);
	}
}
```

and then called during start up.

``` csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory, IApplicationLifetime appLifetime)
{
    ...
    app.ResolveGraphQLSchema();
    ...
}
```

## Summary
- Most container depdendencies are "lazy", only instantiated when required.
- This isn't a problem when there is no logic in their constructors.
- When there is we may see errors or slow downs on first requests.
- Given modern deployment strategies failing fast is a better pattern.
- We can fore evaluation of these dependencies to move to "fail fast" semantics.

[hc]: https://hotchocolate.io/

