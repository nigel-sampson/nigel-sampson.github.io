---
layout: post
title: RSS and Atom Action Results in MVC
tags: mvc csharp
---

One part of MVC I really liked was being able to quickly delegate from the Controller to the View, with all the built in ActionResults the framework has, I could deal with the shape of the data rather than the formatting, especially for results such as JSON.

One thing that all blogs need is to syndicate their content via feeds (usually RSS and / or Atom), .NET has had a lot of support for Syndication built in since .NET 3.5 with System.ServiceModel.Syndication so building someActionResults to use them is pretty easy.

Our RSSResult is pretty simple, we inherit from ActionResult, take a SyndicationFeed as a parameter. On execution we set the content-type of the output, create an RSS formatter for the feed and write it to the Output Stream.

``` csharp
public class RssResult : ActionResult
{
    private readonly SyndicationFeed feed;
 
    public RssResult(SyndicationFeed feed)
    {
        this.feed = feed;   
    }
 
    public override void ExecuteResult(ControllerContext context)
    {
        context.HttpContext.Response.ContentType = "application/rss+xml";
 
        var formatter = new Rss20FeedFormatter(feed);
 
        using(var writer = new XmlTextWriter(context.HttpContext.Response.OutputStream, Encoding.UTF8))
        {
            formatter.WriteTo(writer);
        }
    }
}
```

AtomResult will look pretty much the same, a different content-type and different formatter. Here they are in use.

``` csharp
public ActionResult Syndicate(string format)
{
    var query = new PagingQuery(new PublishedPostsQuery())
    {
        CurrentPageIndex = 0,
        PageSize = 10
    };
 
    var feed = new SyndicationFeed("Compiled Experience", "Silverlight Development", new Uri("http://compiledexperience.com"))
    {
        Items = from p in repository.Find<Post>(query)
                select new SyndicationItem(p.Title, p.Content, new Uri(String.Format("/blog/posts/{0}", p.Slug), UriKind.Relative))
                {
                    PublishDate = p.PostedOn
                }
    };
 
    if(format.Equals("rss", StringComparison.InvariantCultureIgnoreCase))
        return new RssResult(feed);
 
    return new AtomResult(feed);
}
```
