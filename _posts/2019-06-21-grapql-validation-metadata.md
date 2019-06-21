---
layout: post
title: Exposing Validation Metadata in GraphQL
tags: csharp graphql
---

When we do input validation in our applications we want them on both the client and server for a couple of key reasons:

1. Validation on the client introduces better opporutunities for a richer user experience.
2. We validate on the server as well because we can't trust the client.

One other goal we have is to only define our validation rules once, if we have to define our client and server validation in separate locations in our code we run the risk of one location not being updated and having a drift between client and server.

### How is this done in .NET?

ASP.NET has the a [`ModelMetadata`][mm] system that is highly extensible via a provider model and enables a way to view the validation rules. The out of the provider uses the `System.ComponentModel.DataAnnotations` attributes for validation definition giving us something like:

``` csharp
public class CreatePersonInput
{
    [StringLength(100)]
    public string FirstName { get; set;}

    [Required]
    [StringLength(100)]
    public string LastName { get; set;}
}
```

The out of the box Razor view system can then use the model metadata to write these validation rules into the html which can then be used the a front end validation framework such as `jquery-validation-unobtrusive`. You can read more about the whole system in [Model validation in ASP.NET Core MVC and Razor Pages][mv].

### The GraphQL approach

How can reproduce parts of this approach in a GraphQL based system? Our goal is to define validation rules on the input objects that are arguments to our queries and mutations and then have a way to expose those rules to a client consuming our API.

The [GraphQL specification][spec] doesn't have way to define validation rules, but it does have the concept of [directives][dir] which fill much the same space as attributes do in C#. They're a mechanism to annotate a GraphQL schema with metadata that can be consumed by the server, client or tool such as a code generator.

For the example above we may want to have something like the following in our schema:

```
input CreatePersonInput {
    firstName: String
        @stringLength(maximumLength: 100)

    lastName: String!
        @required
        @stringLength(maximumLength: 100)
}
```

> I've wrapped the directives onto the lines after the field, this is just for readaiblity, they can all be on the same line (space separated).

> It's also arguable that the fact that `lastName` is defined as not being nullable and required is redundant, but I've included it here to match up with validation attributes.

### Defining our directives

The first step is create our directives and register them with our schema. I've deliberately kept these simple but you can imagine extending them with custom validation messages and the like. We're building on top of the [Hot Chocolate][hc] GraphQL library.

``` csharp
public class RequiredDirective
{

}

public class RequiredDirectiveType : DirectiveType<RequiredDirective>
{
    protected override void Configure(IDirectiveTypeDescriptor<RequiredDirective> descriptor)
    {
        base.Configure(descriptor);

        descriptor.Name("required");
        descriptor.Location(DirectiveLocation.InputFieldDefinition);
    }
}

public class StringLengthDirective
{
    public int MaximumLength { get; set; }
}

public class StringLengthDirectiveType : DirectiveType<StringLengthDirective>
{
    protected override void Configure(IDirectiveTypeDescriptor<StringLengthDirective> descriptor)
    {
        base.Configure(descriptor);

        descriptor.Name("stringLength");
        descriptor.Location(DirectiveLocation.InputFieldDefinition);
        descriptor.Argument(t => t.MaximumLength)
            .Name("maximumLength")
            .Type<NonNullType<IntType>>();
    }
}
```

``` csharp
var schema = SchemaBuilder.New()
    .RegisterDirective<RequiredDirectiveType>()
    .RegisterDirective<StringLengthDirectiveType>()
    .Create();
```

### Code first schema

In a code first schema approach we would expect to see something like the following for our `CreatePersonInput`, this is just enough configuration to override the default nullable string for `LastName`.

``` csharp
public class CreatePersonInputType : InputObjectType<CreatePersonInput>
{
    protected override void Configure(IInputObjectTypeDescriptor<CreateCommunityMemberInput> descriptor)
    {
        base.Configure(descriptor);

        descriptor.Field(c => c.LastName)
            .Type<NonNullType<StringType>>();
    }
}
```

We could then get the schema outcome by adding the appropriate directives here
``` csharp
    descriptor.Field(c => c.FirstName)
        .Directive(new StringLengthDirective { MaximumLength = 100 });

    descriptor.Field(c => c.LastName)
        .Type<NonNullType<StringType>>()
        .Directive(new RequiredDirective())
        .Directive(new StringLengthDirective { MaximumLength = 100 });
```

This gives us the correct schema, but we've missed on our goal of only definining our validation rules once.

### Using ModelMetadata
The next step is using the model metadata provided to us to automatically discover which validation directives to apply to our schema. Here it's important to understand a bit about how Hot Chocolate creates the schema for out types. First it will call `Configure` above and then infer fields based off properties on the type (this is why we didn't need define `FistName` in the first example above). This means we need a hook point after the inference of the fields to then go through all of the fields and add our directives.  This is called [type extension][extend] in Hot Chocolate and the hook we're looking for is `OnBeforeCreate`.

First we'll create an extension method onto the type descriptor that makes use of the type extension system.

``` csharp
public static IInputObjectTypeDescriptor<T> AddValidationDirectives<T>(this IInputObjectTypeDescriptor<T> descriptor, IModelMetadataProvider metadataProvider)
{
    descriptor
        .Extend()
        .OnBeforeCreate(d => AddValidationDirectives(d, metadataProvider));

    return descriptor;
}
```

and then the meat of the whole approach, we use the metadata provider to find all properties that have validators. We then locate the matching field based on name and attach directives based on which validation attributes are provided.

``` csharp
static void AddValidationDirectives(InputObjectTypeDefinition definition, IModelMetadataProvider metadataProvider)
{
    var metadata = metadataProvider.GetMetadataForType(typeof(CreateCommunityMemberInput));

    foreach (var propertyMetadata in metadata.Properties.Where(p => p.HasValidators ?? false))
    {
        var field = definition.Fields.SingleOrDefault(f => f.Name.Equals(propertyMetadata.Name, StringComparison.OrdinalIgnoreCase));

        if (field == null)
            continue;

        foreach (var validator in propertyMetadata.ValidatorMetadata)
        {
            switch (validator)
            {
                case RequiredAttribute required:
                    field.Directives.Add(new DirectiveDefinition(new RequiredDirective()));
                    break;
                case StringLengthAttribute stringLength:
                    field.Directives.Add(new DirectiveDefinition(new StringLengthDirective
                    {
                        MaximumLength = stringLength.MaximumLength
                    }));

                    break;
            }
        }
    }
}
```

Our schema code then simply becomes

``` csharp
public class CreatePersonInputType : InputObjectType<CreatePersonInput>
{
    readonly IModelMetadataProvider _metadataProvider;

    public CreateCommunityMemberInputType(IModelMetadataProvider metadataProvider)
	{
        _metadataProvider = metadataProvider;
    }

    protected override void Configure(IInputObjectTypeDescriptor<CreateCommunityMemberInput> descriptor)
    {
        base.Configure(descriptor);

        descriptor.Field(c => c.LastName)
            .Type<NonNullType<StringType>>();

        descriptor.AddValidationDirectives(_metadataProvider);
    }
}
```

### Conclusion

We now have in place an extensible approach for defining our validation once per model and then exposing via the GraphQL schema to consuming clients.

Next time we'll look at automatically validating those arguments on the server.

[mm]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelmetadata?view=aspnetcore-2.2
[mv]: https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation?view=aspnetcore-2.2
[spec]: https://graphql.github.io/graphql-spec/June2018/
[dir]: https://graphql.github.io/learn/queries/#directives
[hc]: https://hotchocolate.io/
[extend]: https://hotchocolate.io/docs/extending-types