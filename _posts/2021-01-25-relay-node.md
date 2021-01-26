---
layout: post
title: Implementing GraphQL Relay Node support in Hot Chocolate
tags: csharp graphql
---

## What is Relay?

The [GraphQL spec][spec] isn't very prescriptive on the structure of your schema leaving the design completely in your hands. This means we're likely to see quite a few different approaches to certain patterns between different services. One specification that has become pretty common across services is [Relay][relay]. 

If you check out their webpage you'll notice that it advertises itself with 

> Relay is a JavaScript framework for building data-driven React applications powered by GraphQL, designed from the ground up to be easy to use, extensible and, most of all, performant. Relay accomplishes this with static queries and ahead-of-time code generation.

In order to achieve this [Relay][relay] has to make some assumptions about the structure of your GraphQL service, these are documented at [GraphQL Server Specification][relay-spec]. We won't described it in detail here but ultimately it covers three areas.

1. Being able to fetch arbitrary objects by id.
2. Paging one to many relationships between objects.
3. Mutation inputs and paylods.

For this post we'll cover implementing part of this specification using [Hot Chocolate][hc], in particular the first requirement around arbitrary objects by id.

The [server specific][relay-spec] defines an interface `Node` to designate service objects and being able to accessed globally by id.

``` graphql
interface Node {
    id: ID!
}
```

Objects implementing this interface may look something like

``` graphql
type Product implements Node {
    id: ID!
    name: String!
    category: ProductCategory!
}

type Order implements Node {
    id: ID!
    customer: Customer!
    createdOn: Instant!
}
```

The specification also defines a top level query field that takes an `ID` and returns a `Node`.

``` graphql
type Query {
    node(id: ID!): Node
}
```

This would allow to construct queries that look like

``` graphql
query {
    node(id: "Tm90aWZpY2F0aW9uCmQxNTox") {
        id
        ... on Product {
            name
            category
        }
    }
}
```

It should be obvious from this that the id values for each our nodes should be globally unique.

So what's the value here? This patterns means we may not need to build top level fields to query different objects by id but all can go through this `node` field. In provides standards around identifiers as well.

## Implementing in Hot Chocolate

Our first step will be enabling Relay functionality on our server, this is done during our server setup.

``` csharp
 services
    .AddGraphQLServer()
    .EnableRelaySupport()
    .AddQueryType<QueryType>();
```

Once this is done we can no add `Node` support to any of our types. Let's assume we have a `ProductType` class describing `Product`. 

``` csharp
public class ProductType : ObjectType<Product>
{
     protected override void Configure(IObjectTypeDescriptor<Product> descriptor)
     {
         descriptor
            .ImplementsNode()
            .IdField(n => n.Id)
            .ResolveNode(async (context, id) =>
            {
                var productRepository = context.Service<IProductRepository>();

                return await productRepository.GetById(id);
            });
     }
}
```

Here we're doing three things.

1. Saying that `Product` will implement the `Node` interface.
2. Describe which property on the `Product` will be used for the `Node.id`.
3. A resolver to take the id and return the `Product`.

So what is Hot Chocolate doing here? 

It's adding a field `id` to the `Product` object the resolves to creating a globally unique identifier. It does this by encoding the type information into the id. For instance if internally our product id was the integer `42` then on our GraphQL service the id value will look something like `UHJvZHVjdAppNDI=` which decoded includes the type as well as the id. This also means that when the a query to the `node` field is resolved because the id contains the type information then it can route the query to the resolver in question.

In later posts we'll talk about connections and mutations.


[hc]: https://chillicream.com/
[relay]: https://relay.dev/en/
[spec]: https://spec.graphql.org/
[relay-spec]: https://relay.dev/docs/en/graphql-server-specification.html