---
layout: post
title: Implementing resource based authorization in GraphQL
tags: csharp graphql
---

Late last year I wrote about [Securing a GraphQL endpoint][previous], using [ASP.NET Core policy based authorization][policy]. 

The short version of this is that we can create authorization policies and use [Hot Chocolate][hc] to apply those policies to fields in our schema. If the current user cannot fufill those policy requirements then the execution of that field is stopped and `null` is returned. It's impotant to note that this doesn't stop the entire query but just the field in question.

## What is "resource based authorization"?

All of the examples used in the above articles we can think of "declarative authorization" where able to simply annotate the schema with our authorization requirements. 

Sometimes however we need to take into account the data (or resource) in question needs to be taken into account. Microsoft terms this [resource based authorization][resource] or "imperative authorization". 

Examples of this include:
- Only the author of the document can edit it.
- When viewing customer details if they are a minor then a specific permission is required.

You'll notice that in that article the solution involves adding code to particular MVC actions to enforce thse policies.

## An implementation in GraphQL

### Authorization in the resolver

We can follow a similar implementation model in GraphQL as shown in the above article in MVC. This would be how we add the policy evaluation is the resolver for the field.

``` csharp
public async Task<Document> GetDocument(IResolverContext context, Guid documentId)
{
    // Hot Chocolate pushes the current user into `ContextData`
    var user = (ClaimsPrincipal) context.ContextData[nameof(ClaimsPrincipal)];

    var document = await _repository.Find(documentId);

    var authResult = await _authorizationService.AuthorizeAsync(user, document, "EditPolicy");

    return authResult.Succeeded ? document : null;
}
```

In this code we can reuse the requirement and requirement handlers in [the article][resource] which is quite nice.

### Creating a more reusable policy

We can implement this in another way in [Hot Chocolate 11][hc] which I think puts in a better place, the change they've added is an option to the `@authorize` directive which controls whether the policy evaluation occurs before or after the resolver (the only and now default behavior was before the resolver).  This coupled with the fact that Hot Chocolate injects the `IDirectiveContext` as the reource being evaluated.

We can then modify the requirement handler as follows, here we look at the result of the resolver to get the document and run the same check as we did before.

``` csharp
public class DocumentAuthorizationHandler : 
    AuthorizationHandler<SameAuthorRequirement, Document>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
                                                   SameAuthorRequirement requirement,
                                                   IMiddlewareContext middleware)
    {
        if (middleware.Result is Document document && context.User.Identity?.Name == document.Author)
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }
}
```

We can now apply our "imperative policy" is a declarative fashion as follows.

``` graphql
type Query {
    document(id: ID!): Document @authorize(policy: "EditPolicy", apply: AFTER_RESOLVER)
}
```

One of the major benefits of this approach is I can then easily apply this policy on multiple fields in our schema rather than having to go through our entire code base and inject the code. Even more useful is the ability to apply the directive to types which applies the policy to all fields on the type.

## Conclusion

With the changes in Hot Chocolate 11 controlling when authorization directives are applied can let use build policies that implement "resource based authorization" in way that can be used across the entire schema.

[hc]: https://chillicream.com/
[policy]: https://docs.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-5.0
[resource]: https://docs.microsoft.com/en-us/aspnet/core/security/authorization/resourcebased?view=aspnetcore-5.0
[previous]: https://compiledexperience.com/blog/posts/securing-graphql