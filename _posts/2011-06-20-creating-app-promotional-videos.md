---
layout: post
title: Creating app promotional videos
tags: windows-phone
---

I've received a few emails over the last week on how I created the in app videos for my apps [To Do Today](http://compiledexperience.com/windows-phone-7/to-do) and [Left to Spend](http://compiledexperience.com/windows-phone-7/left-to-spend) so I thought I'd go over the process I used.

To capture the video I used the Expression Encoder Screen Capture tool that comes as part of [Expression Encoder](http://www.microsoft.com/expression/products/Encoder4_Overview.aspx). I then drew out the capture area to just the emulator screen without the chrome. I then ran through the demo of the app (multiple takes due to miss taps), once happy with the recording I then transferred the recording to Expression Encoder itself.

<img src="/content/images/posts/encoder.png" />

I used the following settings for encoding the video, the first time I encoded one I also specified a player which I'd use for all the videos. The settings for the player are included below.

 - **Output Format**: Windows Media
 - **Video**: VC-1 Advanced
 - **Width**: 240
 - **Height**: 400
 - **Video Aspect Ratio**: Source
 - **Resize Mode**: Letterbox
 - **Template**: Expression
 - **Scale Mode**: Stretch to Fill

Now that I have the video and player I need the static chrome for the phone. I pulled this image out of the Metro guidelines [photoshop templates](http://www.microsoft.com/design/toolbox/tutorials/windows-phone-7/photoshop/) and use CSS to wrap the player element with the chrome as a background image.

I'm also using JQuery to embed the Silverlight with a technique in a previous post of mine [Using JQuery to emebed Silverlight](http://compiledexperience.com/blog/posts/using-jquery-to-embed-silverlight).

``` javascript
$(function () {
 
    $("#phone-slider-slides").silverlight("/clientbin/phonemediaplayer.xap", {
        width: 240,
        height: 400,
        enableGPUAcceleration: true,
        initParams: "playerSettings = <Playlist>" +
                        "<AutoLoad>true</AutoLoad>" +
                        "<AutoPlay>true</AutoPlay>" +
                        "<DisplayTimeCode>false</DisplayTimeCode>" +
                        "<EnableCachedComposition>true</EnableCachedComposition>" +
                        "<EnableOffline>true</EnableOffline>" +
                        "<EnablePopOut>true</EnablePopOut>" +
                        "<StretchNonSquarePixels>StretchToFill</StretchNonSquarePixels>" +
                        "<Items>" +
                            "<PlaylistItem>" +
                                "<Title>To Do Today : Compiled Experience : .NET Development in New Zealand</Title>" +
                                "<Height>400</Height>" +
                                "<IsAdaptiveStreaming>false</IsAdaptiveStreaming> " +
                                "<MediaSource>http://compiledexperience.com/content/video/to-do-today.wmv</MediaSource>" +
                                "<VideoCodec>VC1</VideoCodec>" +
                                "<Width>240</Width>" +
                                "<AspectRatioWidth>0.6</AspectRatioWidth>" +
                                "<AspectRatioHeight>1</AspectRatioHeight>" +
                            "</PlaylistItem>" +
                        "</Items>" +
                    "</Playlist>"
    });
});
```

Overall it's a pretty simple process once you have the tools in front of you. Hope this helps people start creating web pages to showcase your Windows Phone apps.
