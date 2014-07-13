---
layout: post
title: Why To Do Today doesn't have a Live Tile
tags: windows-phone to-do-today
---

One of the major feature requests I get for [To Do Today](/windows-phone/to-do) is for Live Tile functionality, I've even received zero reviews because it doesn't have this functionality (that's a whole different issue). Live tiles are one of the major selling points for the Windows Phone 7 and a task list management application lends itself to an updating live tile very easily. It's something I put on my original feature list for version 1.0 before I started building the application, yet after two major updates it hasn't been added. Why?

The short answer is, a very limited API that really holds back what applications can do with live tiles and ties them very closely to your phones data connection. 

Now for the longer more technical explanation:

The functionality I'd been keen to add would be to display the amount of tasks due that day as part of the live tile, possibly highlighting the tile if something is overdue.

A live tile consists of three pieces of information, a title, an image and an count and currently there are two ways to update a live tile, each with their own restrictions and caveats.

### Push Notifications

Push Notifications are where a server run by the application developer sends a notification to Microsoft's notification servers which then send it down to the phone. One flavour of push notification is the tile notification, this lets you update all three parts of the tile relatively easily. 

The major problems with this approach is the use of a third party server owned by the application developer that would store the date required for the notifications, all for the sole purpose of updating the tile on your phone. It would be a massive waste of your data connection shuttling data up to the cloud to send back notifications to update the tile. 

A slightly less data intensive approach is to have the phone send notifications to Microsoft to send back to the phone (sigh), this still has issues for me with the data connection requirement. By this I mean that if your phone doesn't have it's data connection active then the tile won't update which is stupid in my opinion for the required functionality.

### Shell Tile Schedule

This piece of functionality allows you to set up a schedule for the tile to update, the smallest increment is hourly and can happen once off or until its cancelled. This API can only update the live tile image and not the title or count. The image we're updating the tile too must be a remote image so again we're requiring the data connection and will be a static path.

This works well for say weather applications, where you can point people to something like a "New York weather image" and have the schedule refresh the tile to that image each hour. On your own server you'd replace that image with one representing the current weather in New York. Works very well since you're serving up the same image to many users and for data that's been determined by something on the cloud.

This fails to work well for [To Do Today](/windows-phone/to-do) for a number of reasons, the scheduled update isn't granular enough. It can take up to an hour for the tile to update and still requires a data connection.

### What I'd Like To See

A simple API where I can make one off instant updates to all properties of the tile that in no way requires a data connection. That's all that's really required.

One thing I haven't dealt with is dealing with the date, the tile should update on day change to the new amount, realistically this will only be possible with some sort of multi tasking functionality.

At the moment all this functionality is possible as it all mimics features in the built in Calendar application, it's simply not available to the average developer.

I really hope a lot of this changes as they're a great selling point to any application and the phone in general. Let's hope Microsoft lets us make better use of them. 
