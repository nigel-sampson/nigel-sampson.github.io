---
layout: post
title: What's new in Windows 10 - Http Live Streaming
tags: windows-apps windows-phone
---

Microsoft have finally started to show off what's available in Windows 10 for us app developers. Over the next few days we can start to take a look at some of these changes and what it can mean. Today we'll look at what's available for media playback with `AdaptiveMediaSource`.

## Disclaimer

This was written using the initial release of the Windows 10 Developer Tools running on Windows 10 build 10041 and things will mostly change before final release. I'll try to come back and keep these posts up to date as time goes on.


## HTTP Live Streaming

There are a lot of adaptive live video streaming formats out there. The three most popular are:

1. [HLS](http://en.wikipedia.org/wiki/HTTP_Live_Streaming)
2. [MPEG-DASH](http://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP) 
3. [Smooth Streaming](http://www.iis.net/downloads/microsoft/smooth-streaming)

Previously if wanted to use HLS in our Windows 8 apps we needed to use a custom media source, some of the free ones available to download were worth the cost and the better ones were pretty pricey. For the latter two we could use [Player Framework](https://playerframework.codeplex.com/).  

If you're doing any complex media scenarios I'd still recommend [Player Framework](https://playerframework.codeplex.com/) but if you just want to display some HTTP Live Streaming content in your app then there's a lot more in the box with `AdaptiveMediaSource`.

To create an instance of `AdaptiveMediaSource` we use either `CreateFromStreamAsync` or `CreateFromUriAsync`.

``` csharp
var hlsUri = new Uri("http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8");
var hlsSource = await AdaptiveMediaSource.CreateFromUriAsync(hlsUri);
```

This returns a `AdaptiveMediaSourceCreationResult` which includes the source and whether the manifest was able to be downloaded or what error occurred.

``` xml
<MediaElement x:Name="Media" />
```

``` csharp
if (hlsSource.Status == AdaptiveMediaSourceCreationStatus.Success)
	Media.SetMediaStreamSource(hlsSource.MediaSource);
```

One thing to remember is that you need the `internetClient` capability declared in your appxmanifest. Lost some time trying to diagnose a result of `AdaptiveMediaSourceCreationStatus.ManifestDownloadFailure`.

``` xml
<Capabilities>
	<Capability Name="internetClient" />
</Capabilities>
```

## Conclusion
Having this functionality in the box is going to make building any sort of app that is supported by video content a lot easier, can't wait to get to using it.