---
layout: post
title: Specialised LINQ to SQL Repositories
tags: csharp
---


We finished off last time with creating a Repository&lt;T&gt; class and
matching interface. This is a general workhorse of a class that allows
such syntax as<br />

``` csharp
IRepository<Item> repository = new Repository<Item>(new InMemoryUnitOfWork);

IEnumerable<Item> outOfStock = from i in repository where i.QuantityOnHand == 0 select i;

foreach(Item item in repository) { }
```

What I&#39;d like however is to be able to encapsulate some of the more complex
queries behind specialised repositories, these repositories can can
then mocked with tools other than TypeMock and move some great
interaction based testing. We&#39;ll start by moving the out of stock query
above to an ItemRepository

``` csharp
public interface IItemRepository
{
    IEnumerable<Item> GetOutOfStockItems();
}

public class ItemRepository : Repository<Item>, IItemRepository
{
    public ItemRepository(IUnitOfWork unitOfWork)
        : base(unitOfWork)
    {
        
    }

    public IEnumerable<Item> GetOutOfStockItems()
    {
        return outOfStock = from i in this where i.QuantityOnHand == 0 select i;
    }
}
```

The LINQ query does look a little odd since we&#39;re querying the repository
directly, in order to make this a little more readable I add a property

``` csharp
IQueryable<Item> Items
{
    get { return this; }
}
```

to make the LINQ query

``` csharp
from i in Items where i.QuantityOnHand == 0 select i;
```

Interestingly because IItemRepository doesn&#39;t implement
IRepository&lt;Item&gt; we lose externally the ability to execute LINQ
queries against it (this could be desired through). I prefer altering
it however to

``` csharp
public interface IItemRepository : IRepository<Item>
```

in order the use of LINQ and normal repository actions throughout the
system while only refactoring complex or commonly used queries into the
repository itself.

