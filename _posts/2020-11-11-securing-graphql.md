---
layout: post
title: Securing a GraphQL endpoint
tags: csharp graphql
---

One of the first things we'll typically want to do when building out any API whether its' GraphQL or REST is to secure it against users who should not have access to it.

I've seen a few articles about this recently when it comes to GraphQL but these are usually about securing the entire `/graphql` endpoing with a single authentication / authorization policy which isn't very nuanced, especially given the nature of GraphQL.

## Defining requirements

Let's start by definining our requirements for authentication and authorization.

1. A valid JWT is required to be authenticated. This includes validating claims such as issuer and audience of the token.
2. All queries / mutations (including introspection) must require authentication.
3. The JWT token should contain scope claims.
4. Some fields on our schema require specific scopes to access.

## Authentication

Looking at the list above for authentication there's nothing specific to GraphQL here so we should be able to use the out of the box ASP.NET Core functionality that looks something like:

``` csharp
services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options => Configuration.Bind("JwtSettings", options));
```
``` csharp
app.UseAuthentication();
```
With the above we're able to correctly authenticate users with the provided JWT token but we're not yet applying it to any requests. Let's start with fufilling requirement 2.

### Authenticating all queries / mutations

Assuming we're using [Hot Chocolates][hc] code first approach then we'll simply call `Auhorize` with no parameters on the root query type.  By providing no role or policy names we're simply saying the user must be authenticated to access any fields on this type. Since this is the root query type we're not authenticating all queries (including the introspection queries). We can complete a similar addition to the root mutation type.

``` csharp
public class QueryType : ObjectType<Query>
{
	protected override void Configure(IObjectTypeDescriptor<Query> descriptor)
	{
		descriptor.Authorize();

        // field definitions are here
    }
}
```

## Authorization
Now we have working authentication let's work on authorization, for the purposes of this example I'm going to assume our JWT includes some claims called `scope` which are the parts of the schema the authenticated user.  We'll assume our scope values look something like `document:read`, `document:write` and so on.

We're going to be using [policy based authorization][policy], so the first we'll need to do is create some policys that enforce the scopes. In the example below we're defining all our policies upfront, given the number of scopes we're enforcing this is ok but as the list grows it may not be the best approach. In a later post I'll write something up about dynamic policies.

``` csharp
services.AddAuthorization(options => {
	options.AddPolicy("scope:document:read", p => p.RequireClaim("scope", "document:read"));
	options.AddPolicy("scope:document:write", p => p.RequireClaim("scope", "document:write"));
});
```

``` csharp
public class MutationType : ObjectType<Mutation>
{
	protected override void Configure(IObjectTypeDescriptor<Mutation> descriptor)
	{
		descriptor.Authorize();

        descriptor.Field(m => m.CreateDocument(default!))
            .Argument("input", a => a.Type<NonNullType<CreateDocumentInputType>>())
            .Type<DocumentType>()
            .Authorize("scope:document:write");
    }
}
```

In the above code you can see us applying the `@authorize` diretive to the field `createDocument` and specifying the policy to enforce when the mutation is called. 

One question that comes up often is:
> If we're enforcing the policy on the createDocument field what's the point of the authentication on the mutation type?

In the above example the type level directive doesn't add any more security, but it allows to create a "secure by default" position. If another developer who hasn't had their morning coffee comes along adds a new mutation to the type but forgets to add the polcy rules then at least the user but still be authenticated to call the mutation and it's not open to the big bad internet.

### Multiple policies
One of the benefits to using policies and GraphQL is that we don't need to just apply our policies on the root query and mutation fields, but use them across our entire schema.

``` graphql
type Document {
    id: ID!
    title: String!
    history: [DocumentHistory!] @authorize(policy: "scope:document:read:sensitive")
}
```

Here we're apply a policy enforcing a policy requiring a narrower scope to a particular field on the document, so what would happen if a user had the scope `document:read`  but not `document:read:sensitive` and tried to query the history?

They would receive the data they are alloed to access such as id and title, but history would be null and the errors collection of the response would include an error referring to the history field.


[hc]: https://hotchocolate.io/
[policy]: https://docs.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-5.0