---
layout: post
title: Dynamic GraphQL Schemas
tags: csharp graphql
---

## Declaring our schema

When we build a GraphQL service we define our schema, we typically do this via two different mechanisms. The first is often referred to as *schema first* where we declare the schema using the SDL that may look something like:

``` graphql
type Query {
    product(id: ID!): Product
}

type Product {
    id: ID!
    name: String!
}
```

The the other approach is usually referred to as *code first* which will look very different depending on which GraphQL server framework you're using, below is an example from the excellent .NET GraphQL framework [Hot Chocolate][hc].

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

This post isn't going to debate the pros & cons of the two above approaches, they both have strengths and weaknesses but I generally prefer *code first*. There's dozens of articles about this on the internet if you want to read more.

## A surprising benefit of the code first approach

If you have gone and looked at some of the articles discussing the *code first* approach one thing may strike you, it's a very declarative approach, we're using code to create the schema but we're still creating new code when new types come along. The question then becomes, is this always the case?

### Strongly typed explosion

Recently at [Pushpay][pp] I spiked some work on what a Notifications microservice would look like, a simple service that allowed other services to push notifications for users and allow applications to query the notifications for a specific user.

Obviously different notifications will have different metadata associated with that notification based on what sort of notification it is. When an export job has been completed processing then we'll need the id of the export and potentially a link to the result.

It can be very easy when designing the GraphQL schema for this service to try and create a generic schema that can acoomodate all sorts of different types of notifications. Ultimately you'll end up with something that looks like the following:

``` graphql
type Notification
{
    key: NotificationKey!
    createdOn: Instant!
    metadata: [MetadataItem!]!
}

type MetadataItem {
    key: String!
    value: String
}
```

While functional in my opinion it misses the point of GraphQL, why bother trying to build a generic not very type safe model on top of a strongly typed schema?  It's impossible to see from the schema what sort of notifications we receive nor what metadata they'll have. My ideal schema would look something like:

``` graphql
interface Notification {
    key: NotificationKey!
    createdOn: Instant!
}

type ExportCompleted implements Notification {
    key: NotificationKey!
    createdOn: Instant!
    exportKey: String!
    sucessful: Boolean!
}
```

Now this is where the "type explosion" I mentioned in the header for this section comes in. When I first started spiking what a fully strongly typed schema would look like I really had examine what each notification added to the schema it it's entirety and came to the following list. Each notfication type would require (we'll use a complete export for concrete examples):

1. The notification type itself that implements the interface `ExportCompleted`.
2. A mutation field that creates that notification.
``` graphql
createExportCompletedNotification(input: CreateExportCompletedInput!): ExportCompletedResult!
```
3. A input to the above mutation that includes the metadata specific to that notification.
```
input CreateExportCompletedInput {
  exportKey: String!
  sucessful: Boolean!
  # common notification fields such as who it's for here
}
```
4. A result class of that mutation that handles returning the strongly typed mutation payload.
``` graphql
type ExportCompletedCreated {
  notification: ExportCompleted!
}
```
5. A union of the above payload and validation errors (it's a post for another day but we represent validation errors in the schema).
``` graphql
union ExportCompletedResult = ExportCompletedCreated | ValidationErrors
```

Not a huge list but the idea that every developer that wants to add a new notification type has to create five different types and a new mutation isn't attractive, it's high friction and quite a waste of time. 

Rather than using declarative code first can we dynamically create the above types without requireing a mountain of work?

### Defining the definition

First we start with our "source of truth" for nofications, below is a stripped down example of a `NotificationDefinition` class.  This is the declarative piece of the puzzle that provides the data for the type generation below, for your needs it could come from anywhere including a database.


``` csharp
 public class NotificationDefinition
 {
	readonly List<(string, Type, bool)> _data = new List<(string, Type, bool)>();

	public static readonly IReadOnlyCollection<NotificationDefinition> All = new List<NotificationDefinition> {

		new NotificationDefinition("ExportCompleted")
			.WithMetadata<string>("ExportKey", required: true)
			.WithMetadata<bool>("Successful", required: true)
	};

	public NotificationDefinition(string name)
	{
		Name = name.Pascalize();
	}

	public string Name { get; }

	public IReadOnlyCollection<(string, Type, bool)> Metadata => _data;

	public NotificationDefinition WithMetadata<T>(string name, bool required)
	{
		_data.Add((name.Camelize(), typeof(T), required));

		return this;
	}
 }
```

Now instead of trying to declare our types in code we're going to write some code that uses this definition and creates the types for us. This code is going to be very [Hot Chocolate][hc] specific but hopefully the concept applies across other frameworks just as well. Here's we're iterating over our definitions and creating a new `ObjectType`, giving it the correct name, implementing the interface and then adding the correct fields based on the common fields and then the specific fields for that definition. The last step is then registering that type within the schema.

``` csharp
public static ISchemaConfiguration RegisterNotificationTypes(this ISchemaConfiguration configuration, IEnumerable<NotificationDefinition> definitions)
{
    foreach (var definition in definitions)
    {
        var notificationTypeName = definition.Name;

        var notificationType = new ObjectType(d => {

            d.Name(notificationTypeName).Implements<NotificationType>();

            d.Field("key").Type<NonNullType<NotificationKeyType>>().Resolver(c => c.Parent<Notification>().Key);
            d.Field("createdOn").Type<NonNullType<InstantType>>().Resolver(c => c.Parent<Notification>().CreatedOn);

            foreach (var (name, type, required) in definition.Metadata)
            {
                var fieldType = required ? (ITypeNode)
                    new NonNullTypeNode(new NamedTypeNode(type.Name)) :
                    new NamedTypeNode(type.Name);

                d.Field(name)
                    .Type(fieldType)
                    .Resolver(c => c.Parent<Notification>().Metadata[name]);
            }
        });

        configuration.RegisterType(notificationType);
    }

    return configuration;
}
```

We won't show creating all five pieces listed above, most follow similar patterns to the above code. The one interesting one is the mutation field. Here we'll register an `ObjectTypeExtension` that extends our `MutationType` definined elsewhere. This means we can keep the whole definition self contained.

``` csharp
configuration.RegisterType(new ObjectTypeExtension(d => 
    {
		d.Name("Mutation");

		d.Field($"create{notificationTypeName}Notification")
			.Type(new NonNullTypeNode(new NamedTypeNode(mutationResultTypeName)))
			.Argument("input", a => a.Type(new NonNullTypeNode(new NamedTypeNode(mutationInputTypeName))))
			.Resolver(c => c.Resolver<Mutation>().Create(c, definition));
    }));
```

## Summary

A lot of the code above isn't neccesary to understand, the idea we're trying to get across here is that we can use *code first* approach lets us not just use a declarative schema but build ourselves a data driven schema that's built at start up.

[hc]: https://hotchocolate.io/
[pp]: https://pushpay.com