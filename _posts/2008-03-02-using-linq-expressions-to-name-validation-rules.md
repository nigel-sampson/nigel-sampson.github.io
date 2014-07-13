---
layout: post
title: Using LINQ expressions to name Validation Rules
tags: csharp
---

&nbsp;In various places in code we want to use property names in our
code, for instance validation or business rules are quick commonly
named after the property they&#39;re validating. Usually you end up with
code looking a little like this.

``` csharp
Rule<Person> rule = new Rule<Person>("FirstName");
```

As [Ayende](http://www.ayende.com/Blog/archive/2005/10/29/8176.aspx) says &quot;Strings are bad&quot;, there are typo issues and also they&#39;re immune
to refactoring. So here&#39;s a quick way not to use them, I wouldn&#39;t call
this fully fledged as we&#39;re not getting compile time checking yet. The
key part to this code is taking an Expression rather than a Func, this
allows us to examine the expressions in the lambda to get the property
name.

``` csharp
public class Rule<T>
{
    private string name;
 
    public Rule(Expression<Func<T, object>> expression)
    {
        MemberExpression memberExpression = expression.Body as MemberExpression;
 
        name = memberExpression == null ? String.Empty : memberExpression.Member.Name;
    }
 
    public string Name
    {
        get
        {
            return name;
        }
    }
}
```

This gives the syntax for rule creation as follows.

``` csharp
Rule<Person> rule = new Rule<Person>(p => p.FirstName);
```

