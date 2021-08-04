---
layout: post
title: GraphQL Observability
tags: csharp graphql
---

Whenever we have a web application we'll typically want to be able to observe it in production. Typically common questions will be:

- Which endpoints are being exercised the most?
- Which endpoints are running slower than normal?
- Which endpoints are returning errors to our users?

Eagle eyed readers will notice all of these quetsions are about endpoints or routes. Most Application Performance Monotiring (APM) products will have a built in knowledge of HTTP and easily be able to show this to you. [New Relic][nr] which I'll use for the rest of my examples even has an understanding of MVC built in and will classify transactions (requests) using their controller and action names rather than just the url.

### What about GraphQL?
The way a GraphQL server works is that all queries and mutations are POST's to the same url (typically something like `/graphql`) which means you get something like the following in New Relic.

![New Relic without operations](/content/images/posts/new-relic-old.png)

Now this data won't let you answer any of those three questions, and in fact GraphQL comes with even more nuance if you're running a public GraphQL API which we can discuss another time.

### Adding Observability

The [Hot Chocolate][hc] has the concept of a `DiagnosticEventListener`, what we'll do is create our own `NewRelicDiagnosticEventListener` to append some extra information to the current transaction to make it more useful. 

The `DiagnosticEventListener` class has a number of methods you can override depending on what you care about in terms of your diagnostics. For this exacmple we care about `ExecuteRequest`.

``` csharp
public class NewRelicDiagnosticEventListener : DiagnosticEventListener
{
    public override IActivityScope ExecuteRequest(IRequestContext context)
    {
        // implementation goes here
    }
}
```

Now the API pattern for the `DiagnosticEventListener` is an intersting one that I'd never come across before and it's worth a bit of explanation. Typeically for these diagnostic API's we'll need a hook for the start of a request as well one at the end as sometimes you'll want to set up things at the start of the requuest and then collect / use them at the end. This can make a hefty API surface if for every "event" there would need to be two hooks. The [Hot Chocolate][hc] developers resolved this by introducing `IActivityScope` which is simply a rename of `IDisposable`.

``` csharp
public interface IActivityScope : IDisposable
{

}
```

The idea is the framework will call `ExecuteRequest` at the start of the request. We'll then request an `IActivityScope` that will be disposed by the framework at the end of the request.

For our example we'll want to do changes to the New Relic transaction at the end of the request because we'll want to use information parsed from the request body.

Here's our final version, it returns a custom `RequestActvityScope` that on disposale will set the name of the current transaction in New Relic to be the incoming operation name.

``` csharp
public class NewRelicDiagnosticEventListener : DiagnosticEventListener
{
    public override IActivityScope ExecuteRequest(IRequestContext context)
    {
        return new RequestActvityScope(context);
    }

    private class RequestActvityScope : IActivityScope
    {
        private readonly IRequestContext context;
        private bool _isDisposed;

        public RequestActvityScope(IRequestContext context)
        {
            this.context = context;
        }

        public void Dispose()
        {
            if (_isDisposed)
                return;

            NewRelic.Api.Agent.NewRelic.SetTransactionName("GraphQL", context.Operation?.Name?.Value ?? "unknown");

            _isDisposed = true;
        }
    }
}
```

We can then use it with our setup code
``` csharp
services.AddGraphQLServer()
    .AddDiagnosticEventListener<NewRelicDiagnosticEventListener>()
    .AddQueryType<Query>();
```

Now we'll see something more useful in New Relic, all our traffic previously grouped up in `/graphql` has been broken apart into the different operations our front end code is making which makes it a lot easier to answer the questions from the start of the post.

![New Relic with operations](/content/images/posts/new-relic-new.png)

## Conclusion

Out of the box most APM tools will do a poor job of observability into GraphQL operations due to how HTTP is used. We need to do more work on our server to assist the APM to enable us as developers to support our GraphQL API's in production.

[hc]: https://chillicream.com/
[nr]: https://newrelic.com/
