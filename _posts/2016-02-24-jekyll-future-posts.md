---
layout: post
title: Why won't my Jekyll post appear?
tags: jekyll github-pages
---

On February 2nd GitHub Pages (which hosts this website) announced they were [now running with Jekyll 3.0.0][announce] and there were a few changes required mostly around the standardisation on the [kramdown][kd] Markdown engine.

This morning writing the [UWP XAML Behaviors 1.1.0 released][uwp] post I ran into a problem where no matter what I did the post wouldn't show. Other changes to the site were being generated correctly and I wasn't receiving an error email from GitHub so I was reasonably stumped.

Whenever I write a post I tend to date it with local time which depending on the time of day could be a day ahead of the server which hasn't been a problem till now.

In Jekyll 2.x future posts would be published by default, in 3.x they will not, this is outlined in the [Upgrading from 2.x to 3.x][doc] documentation.

In the end I solved this by adding the following to `_config.yml`.

```
future: true
```

I could have also solved this by setting the timezone in configuration as well.

Hope this helps someone.



[announce]: https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0
[kd]: http://kramdown.gettalong.org/
[uwp]: http://compiledexperience.com/blog/posts/behaviors-1.1.0
[doc]: http://jekyllrb.com/docs/upgrading/2-to-3/#future-posts