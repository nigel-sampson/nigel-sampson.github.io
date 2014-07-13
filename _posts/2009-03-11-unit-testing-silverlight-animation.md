---
layout: post
title: Unit Testing Silverlight Animation
tags: csharp silverlight
---

One thing I haven&#39;t really included in any of the example Silverlight
posts are the unit tests. There&#39;s continuous debate among developers
concerning the effectiveness of unit tests, in my opinion they&#39;re
absolutely necessary whenever you&#39;re building an application. Test
driven development (TDD) on the other hand I still find very hard, it&#39;s
a different way to approach development that seems difficult to pick up
without peer programming with someone who already has this approach.

In
my mind the benefit of unit tests comes months down the track when
you&#39;re able to use your suite of unit tests in regression testing. The
most value you can gain is when a bug is located (and there will be a
bug) then the first thing that&#39;s done is build a failing test. Now you
can fix the bug knowing that you haven&#39;t altered any existing expected
behavior and your suite of tests is that much stronger for next time.

Anyway I&#39;m rambling, Silverlight unit testing is certainly a different beast than using NUnit etc. The best way to learn about it is from &quot;[Unit Testing with Silverlight 2](http://www.jeff.wilcox.name/2008/03/silverlight2-unit-testing/)&quot; and &quot;[Asynchronous test support - Silverlight unit test framework and the UI thread](http://www.jeff.wilcox.name/2009/03/asynchronous-testing/)&quot; two excellent articles from Jeff Wilcox.

In this post I&#39;ll run through the tests for AnimationBase and SizeAnimation, there has been some minor refactoring on these classes since they were last shown but nothing major. 

First we want to test the AnimationBase
class, particularly the it creates a storyboard for the element being
animated. Also that if we animate the element multiple times that we
don&#39;t create more than one storyboard. The tests are as follows:

``` csharp
[TestClass]
public class AnimationBaseFixture : SilverlightTest
{
    [TestMethod]
    public void AnimateAddsAStoryboardToResources()
    {
        var animation = new SimpleAnimation();
        var element = new Button();
 
        Assert.AreEqual(0, element.Resources.Count);
 
        animation.Animate(element);
 
        Assert.AreEqual(1, element.Resources.Count);
        Assert.IsInstanceOfType(element.Resources[animation.StoryboardResourceKey], typeof(Storyboard));
    }
 
    [TestMethod]
    public void MultipleAnimationsDoNotAddMoreStoryboards()
    {
        var animation = new SimpleAnimation();
        var element = new Button();
 
        Assert.AreEqual(0, element.Resources.Count);
 
        animation.Animate(element);
        animation.Animate(element);
 
        Assert.AreEqual(1, element.Resources.Count);
    }
 
    [TestMethod]
    [Asynchronous]
    public void FiresAnimationCompletedEvent()
    {
        var animationCompleted = false;
 
        var animation = new SimpleAnimation();
        animation.AnimationCompleted += (s, e) => animationCompleted = true;
 
        var element = new Button();
 
        EnqueueCallback(() => animation.Animate(element));
        EnqueueConditional(() => animationCompleted);
        EnqueueTestComplete();
    }
}
```

Now we need to test the SizeAnimation class. First we&#39;re going to test the simple inputs into the constructor and that exceptions are thrown in the constructor.

``` csharp
[TestMethod]
public void CanConstruct()
{
    var animation = new SizeAnimation(100, 50, TimeSpan.FromSeconds(10));
 
    Assert.AreEqual(100, animation.Width);
    Assert.AreEqual(50, animation.Height);
    Assert.AreEqual(10, animation.Duration.Seconds);
}
 
[TestMethod]
public void CanConstructWithDefaultDuration()
{
    var animation = new SizeAnimation(100, 50);
 
    Assert.AreEqual(AnimationBase.DefaultDuration, animation.Duration);
}
 
[TestMethod]
[ExpectedException(typeof(ArgumentException))]
public void ThrowsExceptionForNegativeWidth()
{
    new SizeAnimation(-10, 50);
}
 
[TestMethod]
[ExpectedException(typeof(ArgumentException))]
public void ThrowsExceptionForNegativeHeight()
{
    new SizeAnimation(100, -50);
}
```

Now
the interesting test, we want to test the size is actually animated. To
do this we need to use the Asynchronous testing from the Silverlight
unit test framework, First we arrange the objects we&#39;ll need, the
animation and a button to actually animate, we also need to know when
the animation is completed so we&#39;ll attach a simple lambda to set a bool when it is. 

Once
the button is created we&#39;ll add it to the test panel and quickly Assert
that it&#39;s at the Width and Height we expect. We&#39;ll thenenque the animation and wait until it&#39;s completed, then we&#39;ll Assert that it&#39;s size is the expected, pretty simple really. 

``` csharp
[TestMethod]
[Asynchronous]
public void AnimatesSize()
{
    var animationCompleted = false;
 
    var animation = new SizeAnimation(100, 50, TimeSpan.FromMilliseconds(10));
    animation.AnimationCompleted += (s, e) => animationCompleted = true;
 
    var element = new Button
                      {
                          Width = 50,
                          Height = 25
                      };
 
    TestPanel.Children.Add(element);
 
    EnqueueCallback(() => Assert.AreEqual(50, element.Width));
    EnqueueCallback(() => Assert.AreEqual(25, element.Height));
 
    EnqueueCallback(() => animation.Animate(element));
 
    EnqueueConditional(() => animationCompleted);
 
    EnqueueCallback(() => Assert.AreEqual(100, element.Width));
    EnqueueCallback(() => Assert.AreEqual(50, element.Height));
 
    EnqueueTestComplete();
}
```

While the Silverlight unit test framework isn&#39;t as easy to get a grips with as something like NUnit given that it needs a browser it&#39;s still very useful and makes it quite simple to test our controls.

