---
layout: post
title: Serialization Gotcha on the Windows Phone 7
tags: csharp windows-phone
---

Almost all Windows Phone 7 applications at some point will store data on to the phone itself, the approach most developers are using is to serialize an object graph to a file which can at next use be simply deserialized. 

It's a nice, easy pattern with few real problems, there's many different ways your objects can be serialized, xml, json or binary are the usual options.

[To Do Today](/windows-phone/to-do) serializes the Tasks, Recurring Tasks and Categories as a single json document using the excellent [Json.Net](http://json.codeplex.com/) library. I was finding however that the performance of the load (deserialization) was degenerating very quickly as the amount of tasks increased, more so than I expected. After some frustrating hours tinkering with no results I jumped into the [profiler](http://www.eqatec.com/profiler/home.aspx) and it became very apparent what my problem was.

Because [To Do Today](/windows-phone/to-do) is a fairly simple application the model classes are used for serialization and exposed by the view models for display purposes. This means they inherit from PropertyChangedBase in Caliburn Micro to provide the INotifyPropertyChanged functionality.

So as each property was populated by the deserializer I was firing a property changed event. The PropertyChangedBase class in Caliburn Micro was also doing some work to dispatch that event back the UI thread since our Load was being performed asynchronously. Twenty tasks with ten properties meant two hundred events being dispatched, a performance nightmare to provide notifications to no one as there certainly wouldn't be any subscribers at that point.

There were two possible solutions to this, I could split the Task into two classes, a simple data storage Task used during the serialization process and another that implemented INPC for the view models. In a large application I would suggest doing this, but for smaller apps it seemed a little overkill.

Thankfully Caliburn Micro has support to turn off the notifications through the IsNotifiying property. So I simply hooked into the deserialization process to turn notifications off while the object is being populated. This resulted in a 75% performance increase in load times.

``` csharp
[OnDeserializing]
public void OnDeserializing(StreamingContext context)
{
    IsNotifying = false;
}
 
[OnDeserialized]
public void OnDeserialized(StreamingContext context)
{
    IsNotifying = true;
}
```

The important lessons here are, always check when you're doing notifications and dispatching events, especially when your objects are being used in different scenarios, secondly profilers are awesome for tracking this stuff down quickly.
