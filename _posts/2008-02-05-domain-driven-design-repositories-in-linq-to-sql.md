---
layout: post
title: Domain Driven Design Repositories in LINQ to SQL
tags: csharp
---

Lately I've been working with NHibernate behind a domain driven design
style repository interface, this allows for easily testability as I can
create mock repositories using Rhino Mocks and give my services
prepackaged results for testing purposes. 

This has worked out great, in combination with NHibernate Query
Generator I get some very readable code. After deciding that my next
pet project would include LINQ to SQL I wanted to achieve the same
results with regard to testability without reliance on an external
database. 

From some reading around the web it looked like doing interaction style
testing on LINQ style queries would be difficult due to extension
methods being static and the like (I realise TypeMock gets around this
due to using the Profiler API rather than dynamic proxies). But at the
minimum I wanted to provide prepackaged results and the ability to
create aggregate repositories with specialised queries that could be
mocked and tested.

This really isn't persistence ignorance as our model is still
tied LINQ to SQL via the attributes, but I still achieve some
worthwhile goals even if I'm not being completely ignorant.

The two best pieces of work I could find on the subject were K. Scott Allen';s "[Trying Out Persistence Ignorance With LINQ](http://odetocode.com/Blogs/scott/archive/2007/08/26/11321.aspx)" and "[Being Ignorant with LINQ to SQL](http://iancooper.spaces.live.com/blog/cns%21844BD2811F9ABE9C%21397.entry)"
by Ian Cooper. I preferred the formers approach the best and really
this work is just wrapping up his work with some personal alterations.

LINQ queries work by a QueryProvider resolving an expression,
LINQ to SQL resolves the expression to SQL whereas something like LINQ
to Amazon resolves the same expression to a web-service call. In order
to achieve our testing goals the QueryProvider that ultimately resolves
the given expression is switched. For our in-memory repository we'll be
using a List<T>; and therefore LINQ to Objects.

We begin by defining out IUnitOfWork, this will create data
sources that will represent either a table in LINQ to SQL or a
List<T> in LINQ to Objects as well as provide a life cycle for
our underlying DataContext. Our InMemoryUnitOfWork is storing it's
created data sources so that repeated creations of a data source will
always have the same underlying list to mimic the behavior of our
SqlUnitOfWork which is always using the same DataContext.

``` csharp
public interface IUnitOfWork : IDisposable
{
    IDataSource<T> GetDataSource<T>() where T : class;
    void SubmitChanges();
}

public class InMemoryUnitOfWork : IUnitOfWork
{
    private Hashtable dataSources = new Hashtable();
    public IDataSource<T> GetDataSource<T>() where T : class
    {
        if (!dataSources.ContainsKey(typeof(T)))
            dataSources.Add(typeof(T), new InMemoryDataSource<T>());
        return dataSources[typeof(T)] as InMemoryDataSource<T>;
    }
    public void SubmitChanges()
    {
    }
    public void Dispose()
    {
    }
}

public class SqlUnitOfWork : IUnitOfWork
{
    private DataContext context;
    public SqlUnitOfWork(DataContext context)
    {
        if (context == null)
            throw new ArgumentNullException("context");
        this.context = context;
    }
    public IDataSource<T> GetDataSource<T>() where T : class
    {
        return new SqlDataSource<T>(context);
    }
    public void SubmitChanges()
    {
        context.SubmitChanges();
    }
    public void Dispose()
    {
        context.Dispose();
    }
}
```

Now to define our IDataSource, it inherits IQueryable&lt;T&gt; to allow
it to be queried using LINQ. Part of the IQueryable&lt;T&gt; interface
is the property Provider, each data source will route all LINQ queries
to their underlying Provider to be appropriately resolved. I&#39;ve also
defined some simple Save, Update and Delete methods, I&#39;m doing this
rather than implementing something like ITable because I want to keep
the repository simple and fairly similar to NHibernate.

``` csharp
public interface IDataSource<T> : IQueryable<T> where T : class
{
    void Update(T entity);
    void Update(IEnumerable<T> entities);
    void Delete(T entity);
    void Delete(IEnumerable<T> entities);
    void Save(T entity);
    void Save(IEnumerable<T> entities);
}
public class InMemoryDataSource<T> : IDataSource<T> where T : class
{
    private List<T> dataSource;

    public InMemoryDataSource()
        : this(new List<T>())
    {
    }

    public InMemoryDataSource(List<T> dataSource)
    {
        if(dataSource == null)
            throw new ArgumentNullException("dataSource");

        this.dataSource = dataSource;
    }

    public IEnumerator<T> GetEnumerator()
    {
        return dataSource.GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    public Type ElementType
    {
        get
        {
            return dataSource.AsQueryable<T>().ElementType;
        }
    }

    public Expression Expression
    {
        get
        {
            return dataSource.AsQueryable<T>().Expression;
        }
    }

    public IQueryProvider Provider
    {
        get
        {
            return dataSource.AsQueryable<T>().Provider;
        }
    }

    public void Delete(T entity)
    {
        dataSource.Remove(entity);
    }
}

public class SqlDataSource<T> : IDataSource<T> where T : class
{
    private Table<T> table;
    
    public SqlDataSource(DataContext context)
    {
        if(context == null)
            throw new ArgumentNullException("context");

        table = context.GetTable<T>();
    }

    public IEnumerator<T> GetEnumerator()
    {
        return table.GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    public Type ElementType
    {
        get
        {
            return table.AsQueryable<T>().ElementType;
        }
    }

    public Expression Expression
    {
        get
        {
            return table.AsQueryable<T>().Expression;
        }
    }

    public IQueryProvider Provider
    {
        get
        {
            return table.AsQueryable<T>().Provider;
        }
    }

    public void Delete(T entity)
    {
        table.DeleteOnSubmit(entity);
    }
}
```

An IRepository&lt;T&gt; is simply another data source but I&#39;ve created
this interface anyway in order to keep everything linked up with domain
driven design concepts and for future extensions. I&#39;ve now also created
a base Repository&lt;T&gt; class that takes a given IUnitOfWork and
then uses it&#39;s given data source for all it&#39;s operations. We can NOW
create a repository with an in memory data source, add items to it
before passing it to services under test. While in our live system use
a LINQ to SQL DataContext to send all our queries to SQL Server.

``` csharp
public interface IRepository<T> : IDataSource<T> where T : class
{
}

public class Repository<T> : IRepository<T> where T : class
{
    private IDataSource<T> dataSource;

    public Repository(IUnitOfWork unitOfWork)
    {
        if(unitOfWork == null)
            throw new ArgumentNullException("unitOfWork");

        dataSource = unitOfWork.GetDataSource<T>();
    }

    public void Update(T entity)
    {
        dataSource.Update(entity);
    }

    public void Update(IEnumerable<T> entities)
    {
        dataSource.Update(entities);
    }

    public void Delete(T entity)
    {
        dataSource.Delete(entity);
    }

    public void Delete(IEnumerable<T> entities)
    {
        dataSource.Delete(entities);
    }

    public void Save(T entity)
    {
        dataSource.Save(entity);
    }

    public void Save(IEnumerable<T> entities)
    {
        dataSource.Save(entities);
    }

    public IEnumerator<T> GetEnumerator()
    {
        return dataSource.GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    public Type ElementType
    {
        get
        {
            return dataSource.ElementType;
        }
    }

    public Expression Expression
    {
        get
        {
            return dataSource.Expression;
        }
    }
    public IQueryProvider Provider
    {
        get
        {
            return dataSource.Provider;
        }
    }
}

[Test]
public void LinqQueryFromInMemoryUnitOfWork()
{
    IRepository<Item> repository = new Repository<Item>(new InMemoryUnitOfWork());

    repository.Save(new Item() { Name = "Item1", QuantityOnHand = 10 });
    repository.Save(new Item() { Name = "Item2", QuantityOnHand = 0 });

    IEnumerable<Item> outOfStockItems = from item in repository where item.QuantityOnHand == 0 select item;

    CollectionAssert.AreCountEqual(1, outOfStockItems.ToList());
}
```

Once I get some appropriate hosting sorted out I&#39;ll post the full code and unit tests.
Next time, creating specialised aggregate repositories...

