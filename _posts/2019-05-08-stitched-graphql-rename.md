---
layout: post
title: Handling name collisions in GraphQL schema stitching
tags: csharp graphql
---

**GraphQL Schema Stitching** is the mechanism of composing multiple GraphQL schemas together into a single unified schema. This plays strongly into the concept of "back-ends for front-ends" which is building a specialised API designed specifically for the app in question. This "API Gateway" takes the schemas of the micro-services needed for it's functionality, combines, extends and exposes them to the front end.

In the .NET ecosystem the framework [Hot Chocolate][hc] has excellent support for stitching together multiple remote schemas as well as customising and extending them. If you're working with GraphQL in this ecosystem I'd recommend checking it out.

## Name collisions
GraphQL has no concept of namespacing and enforces unique names on types, so how should we deal with two different schemas exposing types with the same name?

### Pseudo-namespaces

Some companies have solved this by introducing a namespace concept by prefixing all types in the schema with the service name. The schema for the Orders service may look like the following. While this works and should guarantee that you have no collisions it cam make for some ugly looking schemas.

```
type Orders_LineItem {
    productId; ID!
    quantity: Int!
}

type Orders_Order {
    id: ID!
    items: [Orders_LineItem!]!
}

type Query {
    orders(customerId: ID!): [Orders_Order!]!
}
```

### Renaming

Another approach is to "rename" the type at the stitched schema if your framework supports it, thankfully [Hot Chocolate][hc] does. The `IStitchingBuilder` lets us rename types as we build our new schema. The following snippet stitches our Customers and Orders schemas and renames the `Customer` type in the Orders schema to `CustomerDetails`.

``` csharp
services.AddStitchedSchema(builder => builder
    .AddSchemaFromFile("customers", "./schemas/customers.graphql")
    .AddSchemaFromFile("orders", "./schemas/orders.graphql")
    .RenameType("orders", "Customer", "CustomerDetails")
```

While this works it feels very *stringly typed* and if you have a number of renames to do would become difficult to maintain. How can we do this better?

One extension point provided is `AddMergedDocumentRewriter`, this takes a `Func<DocumentNode, DocumentNode>` and lets you make any customisations and additions to the schema programmatically. So let's see if we can build a way to specify the renames in the schema files rather than in code.

The question of "Why we're using `AddSchemaFromFile` instead of `AddSchemaFromHttp` is a blog post for another day, but it certainlu helps us here.

Our end goal will be to remove the `RenameType` call and instead inside the `orders.graphql` have something like

```
type Customer @rename(name: "CustomerDetails") {
    id: ID!
    firstName: String!
    ...
}
```

## Rewriting the stitched schema

First we should define our `rename` directive, Directives are the mechanism to add metadata to GraphQL schemas much like C# attributes.

``` csharp
public class RenameDirectiveType : DirectiveType
{
    public const string DirectiveName = "rename";
    public const string ArgumentName = "name";

    protected override void Configure(IDirectiveTypeDescriptor descriptor)
    {
        descriptor.Name(DirectiveName);
        descriptor.Location(DirectiveLocation.Object);
        descriptor.Argument(ArgumentName)
            .Type<NonNullType<NameType>>();
    }
}
```

We can now build our document rewriter to use this directive

``` csharp
public static class Rewriters
{
    public static DocumentNode RenameTypes(DocumentNode document)
    {
        var definitions = new List<IDefinitionNode>();

        foreach (var definition in document.Definitions)
        {
            if (definition is ObjectTypeDefinitionNode typeDefinition)
            {
                var renameDirective = typeDefinition.Directives.SingleOrDefault(d => d.Name.Value == RenameDirectiveType.DirectiveName);

                if (renameDirective != null)
                {
                    var newNameArgumment = renameDirective.Arguments.Single(a => a.Name.Value == RenameDirectiveType.ArgumentName);

                    if (newNameArgumment.Value is StringValueNode stringValue) {
                        definitions.Add(typeDefinition.WithName(new NameNode(stringValue.Value)));
                        continue;
                    }

                }
            }

            definitions.Add(definition);
        }

        return document.WithDefinitions(definitions);
    }
}
```

This looks pretty complex but isn't too bad. It's important to note that `DocumentNode` is immutable, so we build a new one with the renamed types, the `WithDefinitions` helper does this, essentially returning a new `DocumentNode` which is the same as the old but with different type definitions.

The rest of the code simply loops over all the type definitions in the document and examines the object type ones closer (if you wanted to support renaming fields this becomes a little more complex), if this object type definition has the rename directive then get the argument value and add the renamed definition to the list.

We can now use our rewriter as follows

``` csharp
services.AddStitchedSchema(builder => builder
    .AddSchemaFromFile("customers", "./schemas/customers.graphql")
    .AddSchemaFromFile("orders", "./schemas/orders.graphql")
    .AddMergedDocumentRewriter(Rewriters.RenameTypes)
```

We can now handle all our renames in the schema files rather than using magic strings in code.

## Summary
- GraphQL schema stiching combines multiple schemas in single exposed schema. 
- Given the lack of namespaces in the specification we need to handle type name collisions between the schema.
- While Hot Chocolate supports renaming types the mechanism leads to a lot of hard to maintain code.
- Building our own document rewriter lets us do directive based renames.

Hope this helps someone.

[hc]: https://hotchocolate.io/