---
layout: post
title: Patch Support in HttpClient
tags: csharp windows-apps windows-phone
---

Lately I've been working with the **System.Net.Http.HttpClient** originally from the WebAPI and part of what's available to build Metro style apps in Windows 8. I really like the way the client has been designed and especially the async /await support.

One thing I noticed while working with the GitHub API was that while HttpClient supports the *PATCH* method there's no nice helper methods like *GetAsync* or *PostAsync*, thankfully it's very easy to put together so here's some extension methods to add **PatchAsync** to HttpClient.

``` csharp
public static class HttpClientExtensions
{
    public async static Task<HttpResponseMessage> PatchAsync(this HttpClient client, string requestUri, HttpContent content)
    {
        var method = new HttpMethod("PATCH");

        var request = new HttpRequestMessage(method, requestUri)
        {
            Content = content
        };

        return await client.SendAsync(request);
    }

    public async static Task<HttpResponseMessage> PatchAsync(this HttpClient client, Uri requestUri, HttpContent content)
    {
        var method = new HttpMethod("PATCH");

        var request = new HttpRequestMessage(method, requestUri)
        {
            Content = content
        };

        return await client.SendAsync(request);
    }

    public async static Task<HttpResponseMessage> PatchAsync(this HttpClient client, string requestUri, HttpContent content, CancellationToken cancellationToken)
    {
        var method = new HttpMethod("PATCH");

        var request = new HttpRequestMessage(method, requestUri)
        {
            Content = content
        };

        return await client.SendAsync(request, cancellationToken);
    }

    public async static Task<HttpResponseMessage> PatchAsync(this HttpClient client, Uri requestUri, HttpContent content, CancellationToken cancellationToken)
    {
        var method = new HttpMethod("PATCH");

        var request = new HttpRequestMessage(method, requestUri)
        {
            Content = content
        };
        
        return await client.SendAsync(request, cancellationToken);
    }
}

```

Hope this helps people.
