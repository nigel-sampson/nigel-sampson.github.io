---
layout: post
title: Refactor your CSS with dot LESS
tags: mvc
---

We all try and maintain clean code, removing redundant and repetitive code, sometimes we even try it with our HTML (I often think of User Controls as Extract Method for ASP.NET) but one area that constantly lets us down is CSS. There's a lot of repetition of constant values, its hard to structure a file well and given the very simple nature of it can be awkward to develop good conventions.

Working in teams can make this worse as multiple people work on single style sheets, lack of conventions and ways to declare and remove repetitious code exacerbate the problem.

So what can we do about it?

As the title suggests I'm looking at a library named [dot LESS](http://dotlesscss.com/), which is a port of the [Less CSS](http://lesscss.org/) library from Ruby. I've found it incredibly easy to setup, registering the appropriate handlers in IIS and web.config and we're away.

I can't design my way out of wet paper bag, so for this rebuild I purchased a theme from the fantastic site [Theme Forest](http://themeforest.net/). With the theme purchased and downloaded one of things I wanted to do was refactor the CSS to use some of the features now available to me with dot LESS.

Stylesheets for most websites are usually packed into one or two monolithic files for performance reasons (less network calls), but this can really inhibit the maintainability as it becomes harder spot duplications or styles that are at cross purposes. What we can do is make use of the @import directive in dot LESS and break apart our larger files into smaller more manageable units and have the compiler pull them back together at the end.

``` css
/* Layout */
 
@import "layout/layout.less";
@import "layout/header.less";
@import "layout/footer.less";
@import "layout/sidebar.less";
@import "layout/breadcrumbs.less";
@import "layout/call-to-action.less";
@import "layout/columns.less";
```

In the end I may have gone too far with this splitting a 2300 line CSS file down to around 25 .less files, but ultimately it became easier to navigate to styles I needed. During the process of breaking these stylesheets down I also added nested styles. This adds some much needed structure to CSS. If you've ever worked with mammoth CSS files they're usually very long, but not that wide, it's often hard to see the relationships between different styles, this new structure allows us to quickly see some of these relationships at a glance. It also removes some unnecessary repetition.

``` css
a
{
    color: #888;
 
    :link, :visited, :hover, :active, :focus
    {
        text-decoration: none;
        outline: none;
        -moz-outline-style: none;
    }
 
    :hover
    {
        color: #aaa;
    }
}
```

Next we'll do is extract commonly used colours to variables, this will make it easier for later changes and eliminate problems where slightly different colours may be used in different parts of the stylesheet.

Once we've done that with colours we should move on to things such as widths and heights. What's really cool here is that we can define some of the internal column widths as computed from a base width. By relating all logical sizes together we can change scale and common margins and paddings and have other areas change to match.

``` css
@column-separator: 48px;
 
@one-fourth-width: 204px;
@one-third-width: 288px;
 
.one_half
{
    width: (@one-fourth-width * 2) + @column-separator;
}
 
.one_third
{
    width: @one-third-width;
}
 
.two_third
{
    width: (@one-third-width * 2) + @column-separator;
}
 
.one_fourth
{
    width: @one-fourth-width;
}
 
.three_fourth
{
    width: (@one-fourth-width * 3) + (@column-separator * 2);
}
```

Having now created new web applications and refactored existing style sheets into dot LESS I must say it's much more useful in new web applications. Existing CSS stylesheets are usually optimised already, however these optimisations take into account the limitations of CSS and therefore it's more time consuming to refactor to dot LESS. I'd suggest for new applications that you start immediately with dot LESS but for existing ones I'd refactor over time (leave the stylesheet a little better each time).
