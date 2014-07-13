---
layout: post
title: Windows Phone 7 Capabilities
tags: windows-phone
---

The capability model in Windows Phone 7 is something I think every developer needs to understand for reasons that I hope become clear. They help define the security sandbox your application runs in on the phone and provide useful information to possible purchasers of your application on the marketplace.

Capabilities are defined as resources of the phone where there could be privacy or security concerns, these include the built in GPS, Data Connection and uniquely identifying the phone or user.

So why are capabilities important? Well beyond telling Microsoft what your application is capable of it's also published to the user in the marketplace. In each application description you'll see **This app makes use of your phone's**: this list is determined by your application manifest file and in my opinion is very important. It's something I look at whenever I consider purchasing an application, if there's a capability listed that isn't a required by a feature and doesn't really have a logical reason for it then in my mind a red flag goes up.

The most common reason for illogical capabilities on applications currently in the marketplace are analytic frameworks. These libraries will make use of the data connection, some will even use unique identification of the user or the phone. Other reasons include the application using some utility libraries that have that capability in them, the functionality may not be used, but you can't tell for sure.

The capabilities of your application are defined in the WMAppManifest.xml file, and by default it's set to all the capabilities.  The reason for this is that if you try and access a resource on the phone where you don't have the capability then you'll receive an exception which can be a bit of pain in the ass when quickly building an application.

``` xml
<Capabilities>
  <Capability Name="ID_CAP_GAMERSERVICES"/>
  <Capability Name="ID_CAP_IDENTITY_DEVICE"/>
  <Capability Name="ID_CAP_IDENTITY_USER"/>
  <Capability Name="ID_CAP_LOCATION"/>
  <Capability Name="ID_CAP_MEDIALIB"/>
  <Capability Name="ID_CAP_MICROPHONE"/>
  <Capability Name="ID_CAP_NETWORKING"/>
  <Capability Name="ID_CAP_PHONEDIALER"/>
  <Capability Name="ID_CAP_PUSH_NOTIFICATION"/>
  <Capability Name="ID_CAP_SENSORS"/>
  <Capability Name="ID_CAP_WEBBROWSERCOMPONENT"/>
</Capabilities>
```

The latest update (October 2010) to the Windows Phone 7 tools provided a console application to detect the actual capabilities used by your application if later you want to edit your WMAppManifest.xml to that, this tool is under *Program Files\Microsoft SDKs\Windows Phone\v7.0\Tools\CapDetect*. Its run using the following:

CapabilityDetection Rules.xml C:\Path\To\MyApp

It's not entirely necessary as the process of submitting your application to the marketplace runs through the same process, Microsoft use static analysis to detect thecapabilities of your application and rewrite the WMAppManfiest.xml file themselves.

You can see the capabilities Microsoft have detected during submission under **Required device capabilities**. Sometimes you may see some capabilities that really don't make sense given what you've written, one thing to remember is that it's not just the capabilities of your application, but of any libraries referenced by your application, so be careful what you reference.

My conclusion is pretty much, be very aware what the capabilities of your application are, and watch out for any your application exposes that may raise red flags for your users.

