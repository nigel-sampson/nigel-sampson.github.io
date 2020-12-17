---
layout: post
title: Custom GraphQL scalars
tags: csharp graphql
---

With the release of [Hot Chocolate 11][hc-11] comes a very slimmed down approach to building custom scalars in GraphQL.

## What are custom scalars?
In GraphQL fields can be complex types such as objects, interfaces and lists or they can be scalar values. The [GraphQL spec][spec] lists all the built in scalars such as `ID`, `Int` and `String`. However we also have the ability to define our own custom scalars, we do this because we want to be able to take advantage of the strong type system that GraphQL provides. With custom scalars we can:

1. Make invalid data for types an error before it even reaches your resolvers.
2. Better express our domain model to clients.
3. Enable client side type generation to be richer.

## What are some examples of custom scalars

One of the standout examples is that the [spec][spec] doesn't support any date / time scalar out of the box, we could use `String` but then we need to ensure it's a valid string everywhere that takes an input and can potentially be harder to enforce a consistent output. While Hot Chocolate does come with a `DateTime` scalar we at Pushpay decided to model a scalar around the `Instant` from [NodaTime][nt] in order to make it every clear about how we're dealing with things like timezones.

Other examples could be for object identifiers, again the [spec][spec] comes with support for `ID` it can have its own problems.  For instance we often see fields with argument signatures that look something like
```
type Query {
    communityMember(organizationID: ID! communityMemberID: ID!): CommunityMember
}
```
It can be far too easy to put the wrong id into the wrong argument when all `ID` types are the same, with different types this becomes a validation error.

## Defining our custom scalar

In Hot Chocolate 10 there was a decent amount of work to build a custom scalar, but now in [v11][hc-11] it's become a lot simpler. Let's take a look at the code for our `Instant` custom scalar:

``` csharp
public class InstantType : ScalarType<Instant, StringValueNode>
{
	public InstantType() : base(nameof(Instant))
	{
	}

	public override IValueNode ParseResult(object? resultValue) =>
		resultValue switch
        {
			Instant i => ParseValue(i),
			null => NullValueNode.Default,
			_ => throw new SerializationException("Could not parse Instant", this)
		};

	protected override Instant ParseLiteral(StringValueNode valueSyntax) => InstantPattern.ExtendedIso.Parse(valueSyntax.Value).Value;

	protected override StringValueNode ParseValue(Instant runtimeValue) => new StringValueNode(InstantPattern.ExtendedIso.Format(runtimeValue));
}
```

We're obviously inheriting from `ScalarType` but what's interesting is the two type arguments, the first is the underlying dotnet type for this scalar often referred to by the framework as the "runtime type". The second argument is how this will be represented in parsed GraphQL. This can sound complex if we imagine passing our scalar as an argument without a variable what might it look like? For our `Instant` and the example below you can see we'll be using a `StringValueNode`, I suspect a majority of custom scalars would be, but things like `IntValueNode` may also make sense.

``` graphql
query {
    payments(after: "020-12-14T01:19:50Z") {
        amount
    }
}
```

In the constructor we pass the name our scalar to the base constructor. This affects the name in the final schema:

``` graphql
scalar Instant
```

From there we define a couple of different parse methods to handle converting between the runtime value and the literal value in a way that should be reasonable obvious.

## Using our new scalar

The best way to use our custom scalar is to add it directly to the schema.

``` csharp
services.AddGraphQLServer()
    .AddType<InstantType>();
```

Now any of our C# types that have `Instant` properties where Hot Chocolate is automatically disovering fields will now use our custom scalar.

``` csharp
public class Payment
{
    public Instant OccurredOn { get; set;}
}
```

``` graphql
type Payment {
    occurredOn: Instant!
}

scalar Instant
```

[hc-11]: https://chillicream.com/blog/2020/11/23/hot-chocolate-11
[spec]: https://spec.graphql.org/June2018/#sec-Scalars
[nt]: https://nodatime.org/