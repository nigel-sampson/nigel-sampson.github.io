---
layout: post
title: Gothcha when creating your own Client Side Data Annotation Validators
tags: mvc csharp
---

<p>For the new version of the website I'm using the Data Annotations validators for validating view models posted to the controllers. As I'm sure everyone who starts down this road does one of the first things I wanted to create was an EmailAttribute by extending RegularExpressionAttribute. The mighty Scott Gu goes through this process in his blog post on [Model Validation](http://weblogs.asp.net/scottgu/archive/2010/01/15/asp-net-mvc-2-model-validation.aspx).

Unfortunately his solution stops just short of everything that's required.

``` csharp
public class EmailAttribute : RegularExpressionAttribute
{
    public EmailAttribute()
        : base(@"\w+([-+.']\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*")
    {
 
    }
 
    public override string FormatErrorMessage(string name)
    {
        return String.Format("{0} is an invalid email address", name);
    }
}
```

The EmailAttribute works correctly on the server but doesn't trigger any validation on the client side. What you need to do is inform the DataAnnotationsModelValidatorProvider (phew) that the EmailAttribute acts in exactly the same way as the RegularExpressionAttribute. This can be achieved with this simple bit of code in Application_Start.

``` csharp
DataAnnotationsModelValidatorProvider.RegisterAdapter(typeof(EmailAttribute), typeof(RegularExpressionAttributeAdapter));
```
