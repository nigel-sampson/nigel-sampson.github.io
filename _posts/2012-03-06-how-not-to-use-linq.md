---
layout: post
title: How not to use LINQ
tags: csharp
---

Since LINQ (Language Integrated Query) was released by Microsoft as part of the .NET Framework 3.5 in 2007 (has it really been that long?) developers have really taken to it, however I often see the same mistakes repeated time and time again and thought I'd cover some of them.

Often developers first encounter LINQ with examples using the query style syntax; this tends to appeal to people initially due to its closeness to SQL in syntax. Only later on do they work with the extension method style syntax (which is actually the one I prefer). This can often lead to syntax like:

``` csharp
var product = products
    .Where(p => p.Id == 42)
    .Select(p => p)
    .FirstOrDefault();

product.UpdatePrice(13.50m);
```

The important point here is that the Select method is a no-op, doesn't do a damn thing and should be removed. Anything that gets in the way of another developer understanding your code should be removed. 

The other method used a lot is FirstOrDefault by developers wanting to turn their collection into a single object.  There's a couple of problems with this, as the name implies all the *OrDefault methods return the default value for the type if they can't return a result. For reference types this means null, so in the code above we'll get a null reference exception on the second line if there aren’t any objects in the collection. 95% of the times I see FirstOrDefault it should have been replaced with First, or to be even more accurate with Single, but I'll get to that.

The important takeaway here is that you should truly think about whether the method may not product a result and if it might not then handle that case. If nothing else then using First will give a more informative exception rather than the more generic null reference one.
The next problem with the code above (assuming Id should be unique) is that we shouldn't be using First but Single, if we're retrieving a product by id then it's certainly an exception if there's more than one! First will happily return the first matching result, Single will throw an exception, so again the takeaway is think about what should be in the collection and whether First or Single is more appropriate. 

This may seem like a small change, but choosing the right method also lets you show your intent to the next developer to maintain this code. If they see FirstOrDefault they may make certain assumptions about how other parts of the system may work.
On a related note all the above methods as well as Any have an overload that takes a predicate so you can remove the redundant Where method and use the overload of Single. Giving us a final refactoring of:

``` csharp
var product = products
    .Single(p => p.Id == 42);

product.UpdatePrice(13.50m);
```

The results of LINQ queries are almost always collections, if there are no results the collection will be empty, the result will not be null. So code like below will never throw the exception.

``` csharp
var onSaleProducts = products
    .Where(p => p.OnSale);

if(onSaleProducts == null)
    throw new ApplicationException("No products on sale");
```

These are the most common mistakes I see with LINQ code, using the wrong method to get the job done, while it may work for testing, once deployed can be very different.

