---
layout: post
title: GraphQL Naming Conventions
tags: csharp graphql
---

The [Hot Chocolate][hc] GraphQL framework does an excellent job building a schema from your C# types by convention. An easy example would be to see a C# type such as:

``` csharp
public class User
{
    public string Username { get; set; }
    public string? Name { get; set; }
    public bool IsVerified { get; set; }
}
```
and it will build a GraphQL object that looks like the following:
``` graphql
type User {
    username: String!
    name: String
    isVerified: Boolean!
}
```

One point to notice is that differences in casing between C# properties and GraphQL field names, this is where we have a difference in naming / casing conventions between the two systems. Normally the out of the box conventions are fine but sometimes we may want to finesse them a bit. This could be because your company already has some conventions agreed upon, or you just plain don't like the out of the box ones.

For us at [Pushpay][pp] one of the things we disagreed was the out of the box enum values convention. By default if you have an enum that looks like:

``` csharp
public enum PaymentMethodType
{
    CreditCard
    ApplePay
    AchBankAccount
}
```
then [Hot Chocolate][hc] will create you a enum that looks like
``` graphql
enum PaymentMethodType {
    CREDITCARD
    APPLEPAY
    ACHBANKACCOUNT
}
```
Now if we look at the GraphQL specification concerning [enums][enum] then it doesn't really make it clear what multi word enum value casing should be. It's only when we look at the enums that represent [directive location][directive] that we see the convention is what we affectionately call "Screaming Snake Case". 

For us one of the other benefits of this change in case is how it affects TypeScript generation based off the schema where `CREDITCARD` becomes `Creditcard` but `CREDIT_CARD` becomes `CreditCard`.

Now we could just go and hard wire these changes into all of our enum types but there's also a chance one may be missed and fixing them after they're in use is a painful topic for another blog post.

Thankfully [Hot Chcolate][hc] has built in support for schema wide customization of naming conventions through the `INamingConventions` interface.

To fix our example above we'll make use of the [Humanizer][hr] library.

``` csharp
public class PushpayConventions : DefaultNamingConventions
{
	public override NameString GetEnumValueName(object value) => value.ToString().Underscore().ToUpperInvariant();
}
```

We simply register this class into our services as normal:

``` csharp
services.AddSingleton<INamingConventions, PushpayNamingConventions>();
```

Now we'll get our "screaming snake case" for enums by default and no chance of one "falling through the cracks.

I'd encourage you all to check out the entire interface and seeing else you may want to customize.

``` csharp
public interface INamingConventions
{
    string GetArgumentDescription(ParameterInfo parameter);
    NameString GetArgumentName(ParameterInfo parameter);
    string GetEnumValueDescription(object value);
    NameString GetEnumValueName(object value);
    string GetMemberDescription(MemberInfo member, MemberKind kind);
    NameString GetMemberName(MemberInfo member, MemberKind kind);
    string GetTypeDescription(Type type, TypeKind kind);
    NameString GetTypeName(Type type);
    NameString GetTypeName(Type type, TypeKind kind);
    bool IsDeprecated(MemberInfo member, out string reason);
    bool IsDeprecated(object value, out string reason);
}
```

[hc]: https://hotchocolate.io/
[hr]: https://github.com/Humanizr/Humanizer
[pp]: https://pushpay.com
[enum]: https://spec.graphql.org/June2018/#sec-Enums
[directive]: https://spec.graphql.org/June2018/#sec-Type-System.Directives