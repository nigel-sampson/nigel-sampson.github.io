---
layout: post
title: Exporting a GraphQL Schema
tags: csharp graphql
---

I've been playing around with [GraphQL][graphql] at work quite a bit lately and starting to put together some thoughts on how it can fit into your application (and more importantly where it doesn't).

One of the things we found we needed everyonce in a while was an export of the schema in the `.graphql` format. Most of the time we were definining our schema in a "code first* way using [GraphQL.NET][gql.net], so we typically didn't have this file to work from.

However a number of tools / frameworks work well if they can import one of these schemas, a good example is schema stiching using [Hot Chocolate][hc].

So how do we export a schema from an already existing [GraphQL.NET][gql.net] service?

Thankfully support comes in terms of the `SchemaPrinter` class. This takes an instance of your `Schema` that you've created by any of the mechanisms the framework supports.

Usage looks something like 

``` csharp
using (var printer = new SchemaPrinter(_schema, new SchemaPrinterOptions {
    IncludeDeprecationReasons = true,
    IncludeDescriptions = true
    })) {

    context.Response.ContentType = "application/text";
    context.Response.StatusCode = (int) HttpStatusCode.OK;

    await context.Response.WriteAsync(printer.Print());

    return;
}
```

I'm using the code in my GraphQL middleware where if no query was posted then we return the schema.

Hope this helps someone.

[graphql]: https://graphql.org/
[gql.net]: https://graphql-dotnet.github.io/
[hc]: https://hotchocolate.io/