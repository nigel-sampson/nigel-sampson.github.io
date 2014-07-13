---
layout: post
title: Implementing an ActiveRecord pattern in Linq to SQL
tags: csharp
---

In earlier posts I talked about about creating Domain Driven Design
style repositories using Linq to SQL. These allowed us to swap between
Linq to SQL repositories and In Memory ones easily (and really anything
that could support IQueryable&lt;T&gt;, this ensured some very nice
testabiliy. In this post I thought I&#39;d go over some it again as well as
a some refactoring I had done to simplfy things, as well as how to
create simple Active Record style data access on top of our
repositories and therefore still ensuring our testability.

Our IUnitOfWork and its implementations (SqlUnitOfWork
and InMemoryUnitOfWork) haven&#39;t changed. However given that
IDataSource&lt;T&gt; and IRepository&lt;T&gt; are essentially the same
I decided to remove IDataSource&lt;T&gt;. Now that we have two
implentations of IRepository&lt;T&gt; (InMemoryRepository and
SqlRepository) we need one that uses these repositories and forms the
base class for our aggregate repositories.

This
base Repository&lt;T&gt; forms the foundation for it all, internally
it&#39;s only really a decorator around the actual specialised repository
that&#39;s accessed from the current Unit Of Work (more about this in a
later post).

``` csharp
public class Repository<T> : IRepository<T> where T : class
{
    public static IRepository<T> Current
    {
        get
        {
            return UnitOfWork.Current.GetRepository<T>();
        }
    }
 
    public Repository()
    {
 
    }
 
    public IQueryable<T> GetAll()
    {
        return Current;
    }
 
    public virtual void Update(T entity)
    {
        Current.Update(entity);
    }
 
    public virtual void Update(IEnumerable<T> entities)
    {
        Current.Update(entities);
    }
 
    public virtual void Delete(T entity)
    {
        Current.Delete(entity);
    }
 
    public virtual void Delete(IEnumerable<T> entities)
    {
        Current.Delete(entities);
    }
 
    public virtual void Save(T entity)
    {
        Current.Save(entity);
    }
 
    public virtual void Save(IEnumerable<T> entities)
    {
        Current.Save(entities);
    }
 
    public IEnumerator<T> GetEnumerator()
    {
        return Current.GetEnumerator();
    }
 
    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
 
    Type IQueryable.ElementType
    {
        get { return Current.ElementType; }
    }
 
    Expression IQueryable.Expression
    {
        get { return Current.Expression; }
    }
 
    IQueryProvider IQueryable.Provider
    {
        get { return Current.Provider; }
    }
}
```

Active Record, this seems to be
the way a lot of people think data access should be, a few static
methods on your classes and away you go. From a testability point of
view I feel repositories certainly work better, but for smaller data
driven applications Active Record can certainly be the way to go.
</p>
<p>
I&#39;d
like to support both camps with this system so lets build on top of
what we already have, first we&#39;ll need a static accessor to the current
repository, we can do that through the unit of work, but since we
already that in our base class we&#39;ll just expose that. Now anytime we
need our repository it can be accessed through
Repositor&lt;T&gt;.Current. This lets us built our ActiveRecordBase.

``` csharp
public abstract class ActiveRecordBase<T> where T : ActiveRecordBase<T>, IIdentifiable
{
    public static IQueryable<T> GetAll()
    {
        return Repository<T>.Current;
    }
 
    public static T GetById(int id)
    {
        return GetAll().Where(i => i.Id == id).FirstOrDefault();
    }
 
    public static IQueryable<T> Find(Expression<Func<T, bool>> predicate)
    {
        return GetAll().Where(predicate);
    }
 
    public static void Save(T entity)
    {
        Repository<T>.Current.Save(entity);
    }
 
    public static void Update(T entity)
    {
        Repository<T>.Current.Update(entity);
    }
 
    public static void Delete(T entity)
    {
        Repository<T>.Current.Delete(entity);
    }
}
```

One important item in there is the IIdentifiable interface, I&#39;ve seen
some posts around either using Dynamic Linq or reflection to implement
the GetById method but I prefer this. This won&#39;t work for all objects
that dont&#39;t have an identity column, but for most of mine it&#39;s fine.
This allows us something really cool, a kinda mix in. A <em>conditional extension method</em> on the repository class.

``` csharp
public interface IIdentifiable
{
    int Id
    {
        get;
    }
}
```

``` csharp
public static T GetById<T>(this IRepository<T> repository, int id) where T : class, IIdentifiable
{
    if(repository == null)
        throw new ArgumentNullException("repository");
 
    return repository.Where(i => i.Id == id).FirstOrDefault();
}
```

``` csharp
public partial class Product : ActiveRecordBase<Product>, IIdentifiable
{
    public static IQueryable<Product> GetOnSaleProducts()
    {
        return GetAll().Where(p => p.IsOnSale);
    }
}
```

Next post I&#39;ll set up a unit test base class for the in-memory tests and managing our unit of work.

