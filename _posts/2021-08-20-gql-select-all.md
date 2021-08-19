---
layout: post
title: Why doesn't GraphQL "SELECT *"
tags: csharp graphql
---

Over the last year or so I've helped onboard a number of engineers at Pushpay to how we do GraphQL. One question that often comes up is

> How can I select all fields on a type?

GraphQL doesn't have the equivalent of `SELECT * FROM Object`, you can only get the fields in the response that are in the query that was sent and in my opinion this is a very good thing. So why is that?

The major reason for me is the ability to make safe breaking changes to a schema. Occasionally (ideally as little as possible) we'll need to make a breaking change to a schema, the way we tend to approach these schema changes is a pair of widening and narrowing changes. To use an exmaple we want to change the type of a field, for instance let's assume we had a type `Payment` with had an `amount` field of type `Decimal`, however now we want to include the curreny as well and it makes sense to tie into the amount. A typical approach would look like:

1. Add new field `amountPaid` to `Payment` with an object type that includes `amount` and `currency`. It's worth nothing that we can't share field names so naming the new field can often be problematic, you may end up with a temporary name and run a second migration to rename the field back to the original. At the same time we can mark the `amount` field with `@deprecated(reason: "Use amountPaid")`.

Now that we have marked the field as deprecated in the schema we can add some functionality to our backend. We've added a `DiagnosticEventListener` that logs any usage of a deprecated field, the include things like API client ids, user agents etc to help track down who is still selecting the deprecated field. This lets use target communication better about how to inform around these changes and monitor ongoing usage of that field.

2. Once we're no longer seeing log messages around usage of the field we can safely remove the field from the schema knowing we won't break functionality of a client.

Now if we had `SELECT *` and our clients were being lazy and decided to use this we wouldn't be able to tell if a client actually cared about the `amount` field, or was just selecting all fields. This would mean we couldn't communicate effectively with our clients nor determine when it was safe to make the breaking change.

All in all a good design decision.