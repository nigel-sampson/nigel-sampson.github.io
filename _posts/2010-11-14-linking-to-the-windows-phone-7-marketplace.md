---
layout: post
title: Linking to the Windows Phone 7 Marketplace
tags: windows-phone
---

So the WP7 has arrived with a launch that's either a success or failure depending on which website you read. I bought mine (an HTC Trophy) a few days ago from Vodafone here in New Zealand and am loving it, while showing it off most people really like the new look for the phone.

And with the phone comes the marketplace, my first test the waters application [To Do Today](/windows-phone/to-do) was accepted last weekend with only a few minor issues, I'm glad the testers caught the problems they did, not really bugs, but missed on some guidelines that makes the application better in my opinion.

So once you have your application in the marketplace you'll want to get the word out to everyone to check out your new cool app. How do we do this?

One of the best ways to get some visibility outside of the marketplace is with a website that links back, same as the "Available on the App Store" buttons you see on iPhone app pages. You can download the standard graphics provided by Microsoft from [Download for Windows Phone 7 Button](http://go.microsoft.com/fwlink/?LinkId=202116). The format for the link is described on MSDN at [How to: Link to Windows Phone Marketplace Content](http://msdn.microsoft.com/en-us/library/ff967553%28v=VS.92%29.aspx). Check out my [To Do Today](/windows-phone/to-do) page for an example.

What about in the app itself? Well if you're using Trial Mode (something I highly reccomend) then you'll want to provide in app buttons that take the user to the purchase page. To be honest it couldn't be simpler.

``` csharp
public void Buy()
{
    var detailTask = new MarketplaceDetailTask();
 
    detailTask.Show();
}
```

Hope this helps someone, get out there and create some apps!
