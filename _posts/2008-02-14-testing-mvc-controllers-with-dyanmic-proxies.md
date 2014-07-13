---
layout: post
title: Testing MVC Controllers with Dyanmic Proxies
tags: mvc csharp
---
So far it looks like the dominant pattern in unit testing MVC
Controllers is via a Test Specific Subclass and for some that&#39;s a pain.
Especially when you&#39;re going to have a lot of controllers, that&#39;s a lot
of the same code you&#39;ll be repeating over and over. So in the principle
of DRY let&#39;s look at using Castle Project&#39;s Dynamic Proxy to do away
with these controller subclasses.

The main principle behind this is creating a simple dynamic proxy
and attaching an interceptor to it. We&#39;ll then intercept calls to
RenderView and RedirectToAction and then present our captured results
back to the user. The first step is creating our proxy which is simply
a called to ProxyGenerator.CreateClassProxy, this is also where we
attach our method interceptor. We pass constructor arguments as well to
provide support for people who want to use dependency injection with
their controllers.

``` csharp
public class ControllerTester<T> where T : Controller
{
    private static ProxyGenerator generator = new ProxyGenerator();
    private T controller;
    private MethodInterceptor interceptor;

    public ControllerTester(params object[] args)
    {
        this.interceptor = new MethodInterceptor();
        this.controller = generator.CreateClassProxy(typeof(T), new IInterceptor[] { interceptor }, args) as T;
    }
    ...
    }
}
```

The only other interesting piece of code here is intercepting the
appropriate methods &quot;RenderView&quot; and &quot;RedirectToAction&quot; and storing
their results.

``` csharp
public void Intercept(IInvocation invocation)
{
    if(invocation.Method.Name == "RenderView")
    {
        renderArgs = new RenderArgs(invocation.Arguments[0].ToString(), invocation.Arguments[1].ToString(), invocation.Arguments[2]);
        return;
    }
    else if(invocation.Method.Name == "RedirectToAction")
    {
        redirectArgs = new RedirectArgs(invocation.Arguments[0]);
        return;
    }

    invocation.Proceed();
}
```

``` csharp
[ControllerAction]
public void ComplexRedirectAction()
{
    RedirectToAction(new { Controller = "cont", Action = "act", Pirates = "Yarr" });
}
```

``` csharp
[Test]
public void RecordsComplexRedirectData()
{
    ControllerTester<StubController> controllerTester = new ControllerTester<StubController>();

    controllerTester.Controller.ComplexRedirectAction();

    Assert.AreEqual("act", controllerTester.RedirectArgs.Action);
    Assert.AreEqual("cont", controllerTester.RedirectArgs.Controller);
    Assert.AreEqual("Yarr", controllerTester.RedirectArgs.Values["Pirates"]);
}
```

However Scott Gu has mentioned that they&#39;ll be making it easier to test
controllers in the next version of the mvc framework. But this is still
a decent pattern if you&#39;re constantly needing a Test Specific Subclass
to override some methods.

