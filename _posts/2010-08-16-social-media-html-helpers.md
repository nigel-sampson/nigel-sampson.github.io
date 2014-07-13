---
layout: post
title: Social Media Html Helpers
tags: mvc csharp
---

I'm currently in the process of writing the next to tutorials in my [Windows Phone 7](/windows-phone) tutorial series and thought I'd share these snippets of code.

``` csharp
public static class SocialMediaExtensions
{
    public static HtmlString DotNetShoutOut(this HtmlHelper htmlHelper, string url)
    {
        url = HttpUtility.UrlEncode(ToAbsolute(htmlHelper.ViewContext.HttpContext.Request, url));
 
        var img = new TagBuilder("img");
 
        img.Attributes["src"] = "http://dotnetshoutout.com/image.axd?url=" + url;
        img.Attributes["alt"] = "Shout It";
        img.Attributes["style"] = "border: 0px;";
        img.Attributes["width"] = "82";
        img.Attributes["height"] = "18";
 
        var a = new TagBuilder("a");
 
        a.Attributes["rev"] = "vote-for";
        a.Attributes["href"] = "http://dotnetshoutout.com/submit?url=" + url;
 
        a.InnerHtml = img.ToString(TagRenderMode.SelfClosing);
 
        return new HtmlString(a.ToString(TagRenderMode.Normal));
    }
 
    public static HtmlString DotNetKicks(this HtmlHelper htmlHelper, string url)
    {
        url = HttpUtility.UrlEncode(ToAbsolute(htmlHelper.ViewContext.HttpContext.Request, url));
 
        var img = new TagBuilder("img");
 
        img.Attributes["src"] = "http://dotnetkicks.com/services/images/kickitimagegenerator.ashx?url=" + url;
        img.Attributes["alt"] = "Kick It";
        img.Attributes["style"] = "border: 0px;";
        img.Attributes["width"] = "82";
        img.Attributes["height"] = "18";
 
        var a = new TagBuilder("a");
 
        a.Attributes["rev"] = "vote-for";
        a.Attributes["href"] = "http://dotnetkicks.com/kick/?url=" + url;
 
        a.InnerHtml = img.ToString(TagRenderMode.SelfClosing);
 
        return new HtmlString(a.ToString(TagRenderMode.Normal));
    }
 
    public static HtmlString Reddit(this HtmlHelper htmlHelper, string url)
    {
        url = HttpUtility.UrlEncode(ToAbsolute(htmlHelper.ViewContext.HttpContext.Request, url));
 
        var img = new TagBuilder("img");
 
        img.Attributes["src"] = "http://reddit.com/static/spreddit7.gif";
        img.Attributes["alt"] = "submit to reddit";
        img.Attributes["style"] = "border: 0px;";
 
        var a = new TagBuilder("a");
 
        a.Attributes["rev"] = "vote-for";
        a.Attributes["href"] = "http://reddit.com/r/programming/submit?url=" + url;
 
        a.InnerHtml = img.ToString(TagRenderMode.SelfClosing);
 
        return new HtmlString(a.ToString(TagRenderMode.Normal));
    }
 
    private static string ToAbsolute(HttpRequestBase request, string url)
    {
        if(request == null)
            throw new ArgumentNullException("request");
 
        if(url == null)
            throw new ArgumentNullException("url");
 
        var format = request.IsSecureConnection ? "https://{0}{1}" : "http://{0}{1}";
 
        return String.Format(format, request.Url.Host, url);
    }
}
```

They're pretty simple little helpers and can be used for any urls. The only thing that may need to change is the ToAbsolute method which is only for the root application. To the use the helpers in my blog I have the following.

``` html
<p>
    <%: Html.DotNetShoutOut(Url.Action("Posts", new { slug = Model.Slug.ToLower() })) %>
    <%: Html.DotNetKicks(Url.Action("Posts", new { slug = Model.Slug.ToLower() })) %>
    <%: Html.Reddit(Url.Action("Posts", new { slug = Model.Slug.ToLower() })) %>
</p>
```
