---
layout: post
title: Automatically validate arguments in GraphQL
tags: csharp graphql
---

In my last post on [Exposing Validation Metadata in GraphQL][validation-metadata] I showed how we can expose validation metadata from the server to the client in order to have to only define our validation rules once. In this post I'll show how to autoamically enforce those validation rules on the server.

Having validation rules defined doesn't mean much if we don't enforce them. In when using MVC this was done through the `ModelState` property on the `Controller`. You'd regularly see code in the following pattern in your controllers.

``` csharp
public IActionResult CreatePerson(CreatePersonInput input)
{
    if (!ModelState.IsValid)
        return View(input);
    ...
}
```

There's nothing wrong with this code, but one thing that did bug me was it didn't feel like the "pit of success".

> The Pit of Success: in stark contrast to a summit, a peak, or a journey across a desert to find victory through many trials and surprises, we want our customers to simply fall into winning practices by using our platform and frameworks. To the extent that we make it easy to get into trouble we fail.

In this case it's too easy to forget to add those lines to your contoller and leave validation compltely off your endpoint. We can solve this in MVC by using filters to enforece validation on all our `POST` requests.

## GraphQL Middleware

Middleware is a very overloaded term when it comes to software development, within the context of frameworks it typically refers to a pluggable pipeline that an operation moves through. What gets confusing is that each framework typically has it's own notion of middleware which can make them confusing when discussing them.

An example of this is that .NET Core has a concept of middleware for any [incoming requests][dn-middleware], in fact [Hot Chocolate][hot-chocolate] the GraphQL framework we're using is implemented as a piece of .NET Core middleware where it examines requests and if it determines that it's a GraphQL query will execute it.

[Hot Chocolate][hot-chocolate] also has it's own [concept of middleware][hc-middleware], in fact it has three. 

- Query Middleware
- Field Middleware
- Directive Middleware

These are all similar in concept in that these are a pipeline that is executed as the GraphQL query / mutation is executed. The difference is simply the scope of the middleware and when it's executed. Let's examine each in turn and in the context of enforcing validation.

### Query Middleware

This middleware is executed once per query / mutation and roughly akin to the MVC filters discussed above. For an MVC system this would be the ideal place for validation, but a GraphQL query can have any number of fields with any number of arguments. Implementing validation in query level middleware would involve having to walk to the document to find all the values and so isn't our best candidate.

### Field Middleware

This middleware is executed whenever the field the middleware is applied to resolved. This gives us a wide range of options. If we apply the field middleware to the schema directly:

``` csharp
SchemaBuilder.New()
    .Use<ValidateInputMiddleware>()
    .Create();
```

then this middleware will execute for every single field on every single request. This may be desirable in some scenarios but for ours is probably overkill. However if we're defining our schema in a code first manner we can apply middleware to just the fields we care about:

``` csharp
descriptor.Field(m => m.CreatePerson(default))
    .Argument("input", a => a.Type<NonNullType<CreatePersonInputType>>())
    .Use<ValidateInputMiddleware>();
```

This middleware will only execute when the `createPerson` mutation is executed which is very limited in scope, but here we're back to having to remember to apply it to the fields we want validation on. The former approach is overly broad but automatic, the latter exactly brroad enough but not automatic.

### Directive Middleware

The latter approach above use a code first schema definition to apply a piece of middleware to only specifc fields. If we're using a schema first approach then how can we do something similar? We can leverage directives, I think of these in the same way as C# attributes, pieces of metadata attached to our queries and schema. In [Hot Chocolate][hot-chocolate] directives can apply middleware. If we define one that looks like:

``` csharp
public class ValidateInputDirective
{

}

public class ValidateInputDirectiveType : DirectiveType<ValidateInputDirective>
{
    protected override void Configure(IDirectiveTypeDescriptor<ValidateInputDirective> descriptor)
    {
        base.Configure(descriptor);

        descriptor
            .Name("validateInput")
            .Location(DirectiveLocation.FieldDefinition)
            .Use<ValidateInputMiddleware>();
    }
}
```

With this defined we can now apply the directive in our schema.

``` graphql
type Mutation {
    createPerson(input: CreatePersonInput!) : Person @validateInput
}
```

## Validate Input Middleware

So now we have three approaches to apply your middleware depending on the how specific or broad you want to apply it, what does the middleware itself look like? In fact it's pretty simple.

``` csharp
public class ValidateInputMiddleware
{
    private readonly FieldDelegate _next;

    public ValidateInputMiddleware(FieldDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(IMiddlewareContext context)
    {
        if (context.FieldSelection.Arguments.Count == 0)
        {
            await _next(context);
            return;
        }

        var errors = context.FieldSelection.Arguments
            .Select(a => context.Argument<object>(a.Name.Value))
            .SelectMany(ValidateObject);

        if (errors.Any()) 
        {
            foreach (var error in errors)
            {
                context.ReportError(ErrorBuilder.New()
                    .SetCode("error.validation")
                    .SetMessage(error.ErrorMessage)
                    .SetExtension("memberNames", error.MemberNames)
                    .AddLocation(context.FieldSelection.Location.Line, context.FieldSelection.Location.Column)
                    .SetPath(context.Path)
                    .Build());
            }

            context.Result = null;

        } else {
            await _next(context);
        }

        IEnumerable<ValidationResult> ValidateObject(object argument)
        {
            var results = new List<ValidationResult>();

            Validator.TryValidateObject(argument, new ValidationContext(argument), results, validateAllProperties: true);

            return results;
        }
    }
}
```

This code is for a field level middleware (regardless of whether it's applied at the schema for field level), if you want to use this as directive middleware simply change the type of the Invoke parmeter from `IMiddlewareContext` to `IDirectiveContext`.

Walking through the code isn't overly complicated, we get the values for all the arguments of the field and validate them combinng the results into a single list. If there are any errors then we report them as errors, set the result of the field to be null, otherwise we we call the next piece of middleware in the chain. This last step is important because it means that the mutation will never be called if there are validation errors.

## Conclusion

In this post we looking at middleware in the context of a GraphQL request and how we can use it to provide automatic validation of arguments. We can see how the middleware can determine to terminate the pipeline to ensure that under the correct circumstances the field is never resolved.


[hot-chocolate]: https://hotchocolate.io/
[hc-middleware]: https://hotchocolate.io/docs/middleware
[dn-middleware]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware
[validation-metadata]: /blog/posts/grapql-validation-metadata