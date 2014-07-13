---
layout: post
title: Simplifying the Composite Specification Pattern
tags: csharp
---

I&#39;ve found the specification pattern a nice way to wrap up a piece
of business rule to be reused through out the application, for instance
in our e-commerce product our promotion entity builds a specification
based of it&#39;s values. We can then use this specification to determine
whether an order is valid for the promotion or even filter lists of
orders using FindAll method on List and so on. Tim McCarthy has a great
article on building Composite Specifications at &quot;[A Composite Specification Pattern Implementation in .NET 2.0</a>&quot;. 

I don&#39;t entirely agree with his approach here as I find the leaf
specifications he has created such as GreaterThanOrEqualToSpecification
in my opinion decrease the readability of the code. While I think it
can be argued that you can use these leaf specifications to be build up
a richer larger actual domain specification it still means that the
creation is difficult to read unless you go about building a fluent
interface for the specification (which I have done in the past). You
then end up with something like the new NUnit fluent interface.

What I&#39;m going to try and achieve here is a way to bring together easily
different domain specifications to create richer ones. The core class
behind this is PredicateSpecification, really this is just a utility
class that combined with C# lambda expressions gives us some
interesting possibilities. 

``` csharp
public class PredicateSpecification<T> : Specification<T>
{
    private Predicate<T> predicate;
 
    public PredicateSpecification(Predicate<T> predicate)
    {
        this.predicate = predicate;
    }
 
    public override bool IsSatisfiedBy(T item)
    {
        return predicate(item);
    }
}
```

For something different I&#39;ve
decided to use the &amp;&amp;, || and ! operators to create our
composite specifications, this goes along with the theme above that in
trying to increase readability we should try and use the readability we
already have, i.e: that most C# developers read x &amp;&amp; y as &quot;x
and y&quot; already so lets not introduce a new concept.

The abstract
Specification class is where we&#39;re adding our new overloaded operators,
and where we use our new PredicateSpecification to combine the two
arguments.

``` csharp
public abstract class Specification<T> : ISpecification<T>
{
    public static Specification<T> operator &(Specification<T> left, Specification<T> right)
    {
        return new PredicateSpecification<T>(t => left.IsSatisfiedBy(t) && right.IsSatisfiedBy(t));
    }
 
    public static Specification<T> operator |(Specification<T> left, Specification<T> right)
    {
        return new PredicateSpecification<T>(t => left.IsSatisfiedBy(t) || right.IsSatisfiedBy(t));
    }
 
    public static Specification<T> operator !(Specification<T> specification)
    {
        return new PredicateSpecification<T>(t => !specification.IsSatisfiedBy(t));
    }
 
    public static bool operator true(Specification<T> specification)
    {
        return false;
    }
 
    public static bool operator false(Specification<T> specification)
    {
        return false;
    }
 
    public abstract bool IsSatisfiedBy(T item);
}
 

And here's some tests to show it working.

[TestFixture]
[TestsOn(typeof(Specification<>))]
public class SpecificationFixture
{
    [Test]
    public void NotOperatorIsOverloaded()
    {
        PredicateSpecification<string> lengthSpecification = new PredicateSpecification<string>(s => s.Length >= 5 && s.Length <= 10);
        ISpecification<string> specification = !lengthSpecification;
 
        Assert.IsTrue(specification.IsSatisfiedBy("a"));
        Assert.IsFalse(specification.IsSatisfiedBy("123456"));
    }
 
    [Test]
    public void AndOperatorIsOverloaded()
    {
        PredicateSpecification<int> lessThan10 = new PredicateSpecification<int>(i => i < 10);
        PredicateSpecification<int> moreThan5 = new PredicateSpecification<int>(i => i > 5);
 
        ISpecification<int> compositeSpecification = lessThan10 && moreThan5;
 
        Assert.IsTrue(compositeSpecification.IsSatisfiedBy(7));
        Assert.IsFalse(compositeSpecification.IsSatisfiedBy(3));
        Assert.IsFalse(compositeSpecification.IsSatisfiedBy(13));
    }
 
    [Test]
    public void OrOperatorIsOverloaded()
    {
        PredicateSpecification<int> moreThan10 = new PredicateSpecification<int>(i => i > 10);
        PredicateSpecification<int> lessThan5 = new PredicateSpecification<int>(i => i < 5);
 
        ISpecification<int> compositeSpecification = lessThan5 || moreThan10;
 
        Assert.IsFalse(compositeSpecification.IsSatisfiedBy(7));
        Assert.IsTrue(compositeSpecification.IsSatisfiedBy(3));
        Assert.IsTrue(compositeSpecification.IsSatisfiedBy(13));
    }
}
```

Let me know what you think.


