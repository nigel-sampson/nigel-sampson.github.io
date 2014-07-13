---
layout: post
title: Entity Framework Repositories
tags: csharp
---

A long time ago I wrote a few posts about using "Domain Driven Design"-esque repositories using Linq to SQL ("[Domain Driven Design Repositories in Linq to SQL](http://compiledexperience.com/blog/posts/domain-driven-design-repositories-in-linq-to-sql)"). I still use that general pattern with a few tweaks with extra layer of a "Unit of Work" to manage context lifetimes. I'm using this rebuild as a chance to play with Entity Framework 4 and so need to implement the appropriate interfaces all over again. The only major functionality change will be bringing in support for Entity Frameworks "Include".

I'm not going to go over the full pattern here, just the new parts for Entity Framework, the IRepository<T> interface has changed that GetAll returns an interface IQuery<T> which is pretty much the IQueryable<T> interface with the Include method.

The implementations for IQuery<T> and IRepository<T> are as follows.

``` csharp
public class EntityRepository<T> : IRepository<T> where T : class
{
    private readonly ObjectSet<T> objectSet;
 
    public EntityRepository(ObjectSet<T> objectSet)
    {
        this.objectSet = objectSet;
    }
 
    public IQuery<T> GetAll()
    {
        return new EntityQuery<T>(objectSet);
    }
 
    public void Save(T entity)
    {
        if (entity == null)
            throw new ArgumentNullException("entity");
 
        objectSet.AddObject(entity);
    }
 
    public void Update(T entity)
    {
        if (entity == null)
            throw new ArgumentNullException("entity");
 
        objectSet.Attach(entity);
    }
 
    public void Delete(T entity)
    {
        if (entity == null)
            throw new ArgumentNullException("entity");
 
        objectSet.DeleteObject(entity);
    }
}

public class EntityQuery<T> : IQuery<T>
{
    private readonly ObjectQuery<T> query;
 
    public EntityQuery(ObjectQuery<T> query)
    {
        this.query = query;
    }
 
    public IQuery<T> Include(string path)
    {
        return new EntityQuery<T>(query.Include(path));
    }
 
    IEnumerator<T> IEnumerable<T>.GetEnumerator()
    {
        return ((IEnumerable<T>)query).GetEnumerator();
    }
 
    IEnumerator IEnumerable.GetEnumerator()
    {
        return ((IEnumerable)query).GetEnumerator();
    }
 
    Expression IQueryable.Expression
    {
        get
        {
            return ((IQueryable)query).Expression;
        }
    }
 
    Type IQueryable.ElementType
    {
        get
        {
            return ((IQueryable)query).ElementType;
        }
    }
 
    IQueryProvider IQueryable.Provider
    {
        get
        {
            return ((IQueryable)query).Provider;
        }
    }
}
```

The actual Repository<T> that gets used by the domain layer simply delegates all it's work back to a internal repository based off the current unit of work. This is important section because it decouples the domain repository from repository doing the actual work and allows me to change the underlying data source if necessary.

``` csharp
public class Repository<T> : IRepository<T> where T : class
{
    private static IRepository<T> Current
    {
        get
        {
            return UnitOfWork.Current.CreateRepository<T>();
        }
    }
 
    public virtual IQuery<T> GetAll()
    {
        return Current.GetAll();
    }
 
    public virtual void Save(T entity)
    {
        if (entity == null)
            throw new ArgumentNullException("entity");
 
        Current.Save(entity);
    }
 
    public virtual void Update(T entity)
    {
        if (entity == null)
            throw new ArgumentNullException("entity");
 
        Current.Update(entity);
    }
 
    public virtual void Delete(T entity)
    {
        if (entity == null)
            throw new ArgumentNullException("entity");
 
        Current.Delete(entity);
    }
}
```

I'll get into the Unit of Work stuff in my next post.
