---
layout: post
title: Adding cross cutting concerns to a GraphQL service
tags: csharp graphql
---

## What are cross cutting concerns?

Typically within a system we can consider something a cross cutting concern if it has to be involved with every action (in this case a request), these things tend to be security, logging, validation and more.

## Using middleware

The idea of a pipeline of middleware that serves every request makes the process of adding some cross cutting concerns to our GraphQL service really quite easy.  The [Hot Chocolate][hc] framework allows you to add "request middleware" that runs for every request, or "field middleware" that runs on every field within that that request. We may use the "request middleware" to ensure basis metrics such as operation name and timings are sent to our observablity system to allow for easier diagbosis of problems. We may use "field middleware" to ensure every argument on a field is correctly validated".  Both of these are done at the service setup.

``` csharp
services.AddGraphQLServer()
    .UseRequest<LoggingMiddleware>() // Adds middleware to every request
    .UseField<ValidationMIddleware>() // Adds middleware to every field in the request
```

One problem with field middleware is that it's a pretty big hammer, we're essentially adding a little bit of code to the resolution of every single field in our schema. For some concerns this may be fine, the scenario we wish to cover is required on 90% of the fields in our schema. However sometimes this may feel unwarranted, we want to add field middleware to a smaller subset of our fields and not go with a blanket approach.

We can target specific fields a few different ways depending on how we're setting up our schema, if we're using a "code first approch" it would look something like the following, here we're being very targeted, applying the middleware to only the fielsd we care about.

``` csharp
public void Configure(IObjectTypeDescriptor descriptor)
{
    descriptor.Field("productSearch")
        .Use<ValidationMIddleware>()
}       // reset of field definition here
```

If you're using a schema first approach it's a bit more complicated, typically the approach I've used before is to declare a directive, attach the middleware to the directive and then apply the directive on the schema where I want the middleware. In fact if you've used the `@authorize` directive from [Hot Chocolate][hc], this is exactly what is happening, we're using the directive to apply a piece of middleware to the field that will enforce security.

There are some risks with this approach though, we're relying on the developer to remember every place we should be putting the middleware as they add new types and fields to our schema, there is a reasonable chance that the middleware would be forgotten. For a cross cutting concern such as security we could have an unsecured field or other incidents.

## Creating a pit of success

As a senior developer I really like thinking about problems and how to make them easier for others on the team.  The phrase "pit of success" is a way to describe this, we want to make the easy, secure, performant approach the easiest to implement, if the easiest path is the best path then we'll "fall" into the correct approach. For our current problem, is there way we can specifically target our middleware without applying it everywhere but not rely on a developer remembering to add it. We can do this with a `TypeInterceptor`, this is a class that we use at schema creation to inject cross cutting concerns in an automatic manner.

The class allows us to inject behavior at different times, for this scenario we want to do it just before the type is completed. We first check to see if we're working with a object type, from there we check each field, if it as any arguments we add our validation middleware.

``` csharp
public class ValidationTypeInterceptor : TypeInterceptor
{
    public override void OnBeforeCompleteType(ITypeCompletionContext completionContext, DefinitionBase? definition, IDictionary<string, object?> contextData)
    {
        if (definition is ObjectTypeDefinition objectDefinition)
        {
            foreach (var fieldDefinition in objectDefinition.Fields)
            {
                if (fieldDefinition.Arguments.Count > 0)
                {
                    fieldDefinition.MiddlewareComponents.Add(FieldClassMiddlewareFactory.Create<ValidationMiddleware>());
                }
            }
        }
    }
}
```

Using our new type interceptor is as easy as the following.

``` csharp
services.AddGraphQLServer()
    .TryAddTypeInterceptor<ValidationTypeInterceptor>()
    ...
```

We've now created our "pit of success", we no longer need to rely on developers to remember to apply the correct middleware at the correct times, we can by convention apply it to the correct places.


[hc]: https://chillicream.com/
