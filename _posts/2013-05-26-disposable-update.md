---
layout: post
title: Update on disposables
tags: csharp
---

Fellow [Marker Metro][mm] dev [Peter Goodman][pete] pointed out that if you're using [Reactive Extensions][rx] then you already have access to a type similar to the DisposableAction class I talked bout in [Disposable Progress Indicators][progress] through a static method [Disposable.Create][disposable].

Using that our Loading method becomes:

``` csharp
protected IDisposable Loading()
{
    IsLoading = true;
 
    return Disposable.Create(() => IsLoading = false);
}
```

[mm]: http://markermetro.com
[pete]: https://twitter.com/petegoo
[rx]: http://msdn.microsoft.com/en-us/data/gg577609.aspx
[progress]: http://compiledexperience.com/blog/posts/disposable-progress
[disposable]: http://msdn.microsoft.com/en-us/library/system.reactive.disposables.disposable.create(v=vs.103).aspx
