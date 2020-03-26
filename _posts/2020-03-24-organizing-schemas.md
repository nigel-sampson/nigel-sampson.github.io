---
layout: post
title: Organizing large schemas in GraphQL
tags: csharp graphql
---

A GraphQL service is made up for two parts, a schema and a collection of resolvers. The approaches for defining the schema will depend a lot on the capabilities library / framework you're using to build the server. In general terms we can broadly categorise in to two categories **schema-first** and **code-first**.

### Schema first
This is where the schema is defined using the SDL (Schema Definition Language) that would look something like the following:

``` graphql
type Query {
    product(id: ID!): Product
}

type Product {
    id: ID!
    name: String!
}
```

This SDL is often declared as a const string in code or in a file that's read in. Schema first is the more common approach you'll see in blog posts and tutorials as it can be easier to follow along.

### Code first
A number of libraries support defining your schema using objects within code, depending on how smart the library is the schema may be able to be inferred from existing classes in your code base. An example using [Hot Chocolate][hc] looks like the following:

``` csharp
public class QueryType : ObjectType<Query>
{
    protected override void Configure(IObjectTypeDescriptor<QueryType> descriptor)
    {
        descriptor.Field(q => q.Product(default!, default!))
            .Argument("id", a => a.Type<NonNullType<IdType>>())
            .Type<ProductType>()
    }
}
```

This approach isn't as easy to get started with as you need to learn the particulars of the library in question. Schema defined as code has some major benefits if you're building a number of GraphQL services and wish to share a common type system between them.

### Other approaches
Thie choice between code first and schema first is not a binary one, other libraries may have different approaches or even allow a mix of approaches. One of the services at [Pushpay][pp] define their schema using a schema first approach but augment it with common types from code.

## Scaling up prolems
All of these approaches have thier pros and cons and it's not the point of this post to debate them. Some of the decision on which approach to use will be subjective as well.

What's really interesting is with each approach how we deal with management of the schema as the service grows in complexity and the amount of types keeps increasing.

### Schema first
If we have our schema defined in a single SDL file than as our types increase the file can grow to a unmanagable size. This leads to merge conflcts as different developers all merge to the same file as well as other problems.

Thankfully most libraries support defining a schema via a `string` as well as a file. This means a very simplistic approach can be to break the schema into smaller files (I'd base the split around DDD aggregate roots). We can then read each file, concatenate the results and use this as our final schema.

This solves most our problems but their are natural points where these types merge, especially at the root types (usually named `Query`, `Mutation` and `Subscription`). You can have multiple definitions of these types across the files and grouping the definition in one file breaks the idea of each file being relatively self contained.

Thankfully GraphQL has the `extend` keyword we can use to solve this problem. We may start with a `shell.gql` file that defines an empty `Query` type.

``` graphql
type Query {

}
```
Following that our `products.gql` can then extend `Query` and add new fields.
``` graphql
extend type Query {
    product(id: ID!): Product
}
```
This gives use nice self contained subsections of our schema that can then be combined to build a full schema.

### Code first
The code first approach suffers less from the problems as scale as most developers naturally break their classes into separate files. If you have a lot of types you may want break them up into namespaces / folders along the same lines as I described in the schema first approach.

And just like schema first we have the same collision points around the `Query`, `Mutation` and `Subscription` types. Whether the same solution works will depend a bit on the library, thenkfully [Hot Chocolate][hc] supports defining object extensions in code first. We may start with a empty `Query` type

``` csharp
public class QueryType : ObjectType<Query>
{

}
```

and then use extension types in the namespaces
``` csharp
public class ProductsQueryTypeExtension : ObjectTypeExtension
{
    protected override void Configure(IObjectTypeDescriptor<QueryType> descriptor)
    {
        descriptor.Name(nameof(Query));
        descriptor.Field(q => q.Product(default!, default!))
            .Argument("id", a => a.Type<NonNullType<IdType>>())
            .Type<ProductType>()
    }
}
```
## Conclusion
Regardless of the approach we use to define the schema using extensions allows us to break up the schema in ways that will become manageable throughout your development.

[hc]: https://hotchocolate.io/
[pp]: https://pushpay.com