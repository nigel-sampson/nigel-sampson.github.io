---
layout: post
title: Creating and enforcing GraphQL schema conventions
tags: csharp graphql
---

When we're building an API we're creating a contract between ourselves and our consumers. We should ideally make that contract as easier to understand as possible, this can help reduce support burden on the API team and empower the consumer to do more with the API.

GraphQL comes with a strong type system that represents the fields available to query and their types. Collectively this collection of types is known as the **schema** of the service.

Typically when design the schema we'll have a series of conventions or rules for the shape of the schema.  Examples of these types of conventions could be:

### Naming
- Type names should be pascal cased.
- Field names should be camel cased.
- Arguments should be camel cased.
- Interfaces should not have the `I` prefix from donet.
- Types shouldn't have the `Type` suffix.

### Structure
- All enums should support the `UNKNOWN` value.

### Security
- All root level mutations are secured against unauthenticated access.

It's worth nothing that a lot of these conventions may be enforced by your GraphQL framework of choice.

## Enforcing conventions

Unit tests are probably the easiest way to create and enforce conventions against our schema. If we can express the convention in code then we can easily determine if parts of our schema don't conform to the convention.

The first thing our tests will need to do is create the same schema as our application.

``` csharp
// This method is the schema setup code that would likely be called in ASP.NET Startup
static void CreateSchema(IServiceCollection services)
{
    services.AddGraphQL(
        SchemaBuilder.New()
            .AddQueryType<QueryType>()
            ... // more setup code here
        );
}

static ISchema CreateSchemaForTesting()
{
	var services = new ServiceCollection();

	CreateSchema(services);

	var provider = services.BuildServiceProvider();

	return provider.GetRequiredService<ISchema>();
}
```

With these helper methods in place we can create tests that use linq to expore our collection of types. The following ensures all our enums support `UNKNOWN`. One of the reasons we do this is to esnsure the uninitialized variables of database columns don't automatically hold a business value.

``` csharp
[Fact]
public void EnumTypesShouldSupportUnknown()
{
	var schema = CreateSchemaForTesting();

	var enumsWithoutUnknown = schema.Types
		.OfType<EnumType>()
		.Where(t => !IntrospectionTypes.IsIntrospectionType(t.Name))
		.Where(t => t.Values.All(v => v.Name != "UNKNOWN"))
		.ToList();

	Assert.Empty(enumsWithoutUnknown);
}
```

One important thing we want to do is remove the introspection types from the mix, otherwise we'd see failures for the enums `__DirectiveLocation` and `__TypeKind`.

### Adding conventions to an existing schema

Obviously the earlier in a project's lifecycle we can apply rules and conventions like these the better. This enables us to ensure all our future changes to the schema are inline with our intended direction. Sometimes however this isn't the case. We may have an existing schema we want to provide a stronger future direction to by using convention based tests such as these.

The problem comes when we may have parts of our existing schema that violate our conventions. Ideally we'd update our schema to reflect our conventions but if this is a schema that's being used in production these updates represent breaking changes to our clients. We may have to go through a migration process to get our clients using the more conventional approach. This would look something like:

1. Add the new types / fields to the schema, but preserve the existing non-conventional types / fields, mark them as `@deprecrated`.
2. Use stats / metrics to determine when none of your clients are using the deprecated types / fields any more.
3. Remove the deprecated types / fields.

When we're going through this process we still want to have our conventions enforced for new changes but we capture these existing types as "exceptions to the rule" until we've managed to migrate our users away from them. 

This is where [Approval Tests](https://compiledexperience.com/blog/posts/approval-tests) come in, if we update our test to replace the `Assert` with the following we can capture all the existing broken conventions in our approval file. 

``` csharp
Approvals.VerifyAll("All enums should support an UNKNOWN value.", enumsWithoutUnknown, t => t.Name);
```

New types that break the convention will cause the approval to fail and as we work through the migration we can update the approval file. Once we have no existing types against the convention we can update the test to our `Assert`.

## Conclusion
We can use unit tests to explore our GraphQL schema to ensure it follows conventions and approaches we've decided up when designing our schema.

We can then use tools such as apporval tests to capture existsing broken conventions when applying these rules to existing schemas while working to remove them.