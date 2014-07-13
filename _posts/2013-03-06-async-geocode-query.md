---
layout: post
title: Turning Geocode Query into a proper async operation
tags: csharp windows-phone
---

The Windows Phone 8 platform comes with some very useful services around Maps and Location, these lurk in the Microsoft.Phone.Maps.Services namespace of the Microsoft.Phone.Maps assembly. There are three services available encapsulated by the classes [GeocodeQuery][geocode], [ReverseGeocodeQuery][reversegeocode] and [RouteQuery][route] with self-explanatory purposes.

However they are a little weird, despite being part of Windows Phone 8 and therefore having access to the new async features these queries use a QueryCompleted event and then a QueryAsync method to trigger it. You can see some examples of them being used on [Nokia’s Developer Wiki][nokia].

Thankfully we can write a very simple extension method to provide us a way to await these queries, and written in a generic manner so that the same method will work for all three.

We’re going to use a similar approach to how we [awaited storyboard completion][async] in using TaskCompletionSource. In fact the approach is almost identical, we create a TaskCompletionSource and return its Task property. At the same time we use a self-removing event handler attached to the QueryCompleted event of the Query.

``` csharp
public static Task<IList<MapLocation>> ExecuteAsync(this GeocodeQuery query)
{
    var taskSource = new TaskCompletionSource<IList<MapLocation>>();
 
    EventHandler<QueryCompletedEventArgs<IList<MapLocation>>> handler = null;
 
    handler = (s, e) =>
    {
        query.QueryCompleted -= handler;
 
        if (e.Cancelled)
            taskSource.SetCanceled();
        else if (e.Error != null)
            taskSource.SetException(e.Error);
        else
            taskSource.SetResult(e.Result);
    };
 
    query.QueryCompleted += handler;
 
    query.QueryAsync();
 
    return taskSource.Task;
}
```

The only major difference is we’re going to make use of more than SetResult, Queries can be cancelled and they can fail due to things like no data connection. To push these changes into the Task we use the SetCancelled and SetException methods. For the method that’s awaiting this method Cancellation will surface as a TaskCancelledException while the Exception will be thrown.

Now that we’ve written the method for GeocodeQuery we could go and duplicate this with the two other query classes. Fortunately GeocodeQuery inherits Query<T> which exposes everything we need to write the generic version to work with all three (and any new ones).

``` csharp
public static Task<T> ExecuteAsync<T>(this Query<T> query)
{
    var taskSource = new TaskCompletionSource<T>();
 
    EventHandler<QueryCompletedEventArgs<T>> handler = null;
 
    handler = (s, e) =>
    {
        query.QueryCompleted -= handler;
 
        if (e.Cancelled)
            taskSource.SetCanceled();
        else if (e.Error != null)
            taskSource.SetException(e.Error);
        else
            taskSource.SetResult(e.Result);
    };
 
    query.QueryCompleted += handler;
 
    query.QueryAsync();
 
    return taskSource.Task;
}
```

We can now call GeocodeQuery in a much more succinct fashsion.

``` csharp
var geocodeQuery = new GeocodeQuery { SearchTerm = "46 Anglesea St, Auckland" };
 
var locations = await geocodeQuery.ExecuteAsync();
```

[async]: http://compiledexperience.com/blog/posts/awaiting-storyboard-completion
[nokia]: http://www.developer.nokia.com/Community/Wiki/What's_new_in_Windows_Phone_8#Maps
[geocode]: http://msdn.microsoft.com/en-US/library/windowsphone/develop/microsoft.phone.maps.services.geocodequery(v=vs.105).aspx
[reversegeocode]: http://msdn.microsoft.com/en-us/library/windowsphone/develop/microsoft.phone.maps.services.reversegeocodequery(v=vs.105).aspx
[route]: http://msdn.microsoft.com/en-us/library/windowsphone/develop/microsoft.phone.maps.services.routequery(v=vs.105).aspx
