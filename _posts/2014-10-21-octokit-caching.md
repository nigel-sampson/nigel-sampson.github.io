---
layout: post
title: Octokit.Caching
tags: octokit-caching windows-apps windows-phone
---

A few months ago I tweeted about a new open source project [**Octokit.Caching**][oct.cach] but completely forgot do any more announcing.

<blockquote class="twitter-tweet" lang="en"><p>Simple etag caching for Octokit <a href="https://t.co/il9asLfZkJ">https://t.co/il9asLfZkJ</a> cc <a href="https://twitter.com/shiftkey">@shiftkey</a> <a href="https://twitter.com/haacked">@haacked</a></p>&mdash; Nigel Sampson (@nigelsampson) <a href="https://twitter.com/nigelsampson/status/485379375550849025">July 5, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### What does it do?

It's a compact library designed to add simple caching to [Octokit.NET][oct] a GitHub API client library for .NET. It maintains an internal cache of all requests and responses to and from the GitHub API. 

If you make a request where a cached response is already available then it appends the `If-None-Match` header with the cached [etag][etag].

If the server returns a `304 Not Modified` then we return the cached response, otherwise we return the new response while modifying the cache.

### Where's the cache?

[Octokit.Caching][oct.cach] comes with an interface `ICache` and a default implementation `NaiveInMemoryCache`

``` csharp
public interface ICache
{
    Task<T> GetAsync<T>(string key);
    Task SetAsync<T>(string key, T value);

	Task ClearAsync();
}
``` 

It's built for you to extend easily with something like [Akavache][ak].

### Where can I get it?

The source is available on [Github][oct.cach] and is available on [nuget][ng].

[oct.cach]: https://github.com/nigel-sampson/octokit.caching
[oct]: https://github.com/octokit/octokit.net
[etag]: http://en.wikipedia.org/wiki/HTTP_ETag
[ak]: https://github.com/akavache/Akavache
[ng]: https://www.nuget.org/packages/Octokit.Caching/