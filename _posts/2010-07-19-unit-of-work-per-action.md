---
layout: post
title: Unit of Work per Action
tags: mvc csharp
---

Most of the time I do any database interaction I follow a "Unit of Work" pattern. This allows good control over when work is submitted to the database, in terms of the Entity Framework, the Unit of Work controls the lifetime of the ObjectContext.

The implementation is pretty simple, we have a UnitOfWork static class that provides easy access to the current UOW as well as starting and disposing. We then have our IUnitOfWork which is where the work of accessing repositories and submitting changes to the database will be done. We then implement that with EntityUnitOfWork.

I like to follow the approach of "Unit of Work per Request" for web applications and usually handle this with an IHttpModule that manages it for me. However for MVC I wanted to try something different. One of the reasons I didn't like Module approach for MVC was that the UnitOfWork was still active during the rendering of the View. This meant that any complex lazy loaded queries could still be evaluated which could leave me vulnerable to N+1 problems.

``` csharp
public class UnitOfWorkAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        UnitOfWork.Start();
    }
 
    public override void OnActionExecuted(ActionExecutedContext filterContext)
    {
        if(UnitOfWork.IsStarted)
            UnitOfWork.Dispose();
    }
}
```

In the end I created a UnitOfWorkAttribute which inherits from ActionFilterAttribute, this enables me to dispose of the current unit of work after the action is completed but before the view is processed. It means that if the view triggers anything that causes database access it throws an error. I can now enforce the convention that all data must be loaded by the action.

``` csharp
[HttpPost, UnitOfWork, ActionName("sign-in")]
public ActionResult SignIn(SignInViewModel details, string returnUrl)
```

