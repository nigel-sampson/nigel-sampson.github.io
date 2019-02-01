---
layout: post
title: EF Core Client Side evaluation (and how to stop it)
tags: csharp ef
---

EF Core has a feature that supports parts of a query being evaluated on the server and parts on the client, the decision is driven by the whether the underlying LINQ provider can convert the expression into SQL.

An example would be if we had a method like the following:


``` csharp
public static decimal CalculateTax(decimal price) => price * 0.15m;
```

and we used it in the query that looked like 

``` csharp
var products = context.Products
    .OrderBy(p => p.Price)
    .Select(p => new
    {
        p.Id,
        p.Price,
        Tax = CalculateTax(p.Price)
    });
```

the LINQ provider knows how to convert the `OrderBy` clause to SQL so that will be run on the server, but it has no idea how to convert the `CalculateTax` method to SQL so the `Select` clause will be run on the client. From the point of view of the developer unless you're looking very carefully at your query it's not immediately apparent what will run where.

Consider the following query:

``` csharp
var products = context.Products
    .Where(p => CalculateTax(p.Price) > 100.0m);
```

Again we can't evaulate `CalculateTax` server side so it's evalated client side. For this to happen we've had to pull the entire contents of the Products table into memory! If this table is of a significant size then the performance problems in terms of memory and time are going to be really nasty.

Now both of the above queries are pretty simple, it's relatively easy to determine what the client / server execution breakdown will look like, however as queries become more complex this task becomes harder and you run the risk of introducing nasty performence regressions.

EF Core will log when it drops from server to client evaluation it will log this occurance, but it can be easy to miss.

To sum this feature up, I'd avoid it like the plague. Client side evaluation makes it too easy to write a query that has unintended performance regressions without noticing until it's till late.

In my opinion it's better to be explicit on defining where the query happens. But the best first step is disabling client side evaluation, the following code on the `DbContext` changes the drop from server to client from a log warning to an exception. 

``` csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .ConfigureWarnings(w => w.Throw(RelationalEventId.QueryClientEvaluationWarning));
}
```

This means that whenever you write a query that can't fully be evaluated in SQL an exception will be thrown and we'll be fully aware of our problem (at development time).

Revisiting our earlier query that would now throw an exception would need to be written to something like in order to not throw that exception.

``` csharp
var products = context.Products.ToList();

products = products.Where(p => CalculateTax(p.Price) > 100.0m).ToList();
```

It's now very clear what is being evaulated server side and what's on the client side and hopefully the code smell of the entire Products table being loaded into memory is very apparent.

In short, disable this feature as the first thing you do in order to not shoot yourself in the foot.

Hope this helps.
