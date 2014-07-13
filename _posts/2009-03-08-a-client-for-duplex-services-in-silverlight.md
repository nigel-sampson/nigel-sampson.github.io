---
layout: post
title: A client for duplex services in Silverlight
tags: wcf csharp silverlight
---

I&#39;m currently working on a project that requires the use of duplex
services to report progress on a long running event (I&#39;ll go more into
the actual event in a later post). The reference material I started
with was &quot;[How to: Build a Duplex Service](http://msdn.microsoft.com/en-us/library/cc645027%28VS.95%29.aspx)&quot; and &quot;[How to: Access a Duplex Service with the Channel Model](http://msdn.microsoft.com/en-us/library/cc645028%28VS.95%29.aspx)&quot;,
building the service itself its a relatively simple affair with a
slight learning experience as I hadn&#39;t really used the Message class
from WCF before.

The client is relatively simple as well,
provided you can work your way through the copious amounts of
asynchronous methods. Essentially all we&#39;re doing is opening a factory,
then a channel and creating a receive message loop. This code looks
really boilerplate so I encapsulated it into a nicely reusable
DuplexServiceClient.

``` csharp
public class DuplexServiceClient
{
    private IChannelFactory<IDuplexSessionChannel> factory;
    private readonly Dictionary<string, IMessageHandler> handlers;
    private readonly string serviceUrl;
    private readonly SynchronizationContext uiThread = SynchronizationContext.Current;
    private IDuplexSessionChannel channel;
 
    public DuplexServiceClient(string serviceUrl)
    {
        this.serviceUrl = serviceUrl;
        handlers = new Dictionary<string, IMessageHandler>();
    }
 
    public void Open()
    {
        var binding = new PollingDuplexHttpBinding
        {
            InactivityTimeout = TimeSpan.FromMinutes(1)
        };
 
        factory = binding.BuildChannelFactory<IDuplexSessionChannel>(new BindingParameterCollection());
 
        var openResult = factory.BeginOpen(r =>
        {
            if(!r.CompletedSynchronously)
                CompleteOpenFactory(r);
        }, factory);
 
        if(openResult.CompletedSynchronously)
            CompleteOpenFactory(openResult);
    }
 
    public event EventHandler<MessageReceivedEventArgs> MessageReceived;
 
    protected virtual void OnMessegeReceived(MessageReceivedEventArgs e)
    {
        var messageReceived = MessageReceived;
 
        if(messageReceived != null)
            messageReceived(this, e);
    }
 
    public void SendMessage(string action, object data)
    {
        var message = Message.CreateMessage(MessageVersion.Soap11, action, data);
 
        var result = channel.BeginSend(message, r =>
        {
            if(!r.CompletedSynchronously)
                CompleteSend(r);
        }, channel);
 
        if(result.CompletedSynchronously)
            CompleteSend(result);
    }
 
    private void CompleteSend(IAsyncResult result)
    {
        channel.EndSend(result);
    }
 
    private void CompleteOpenFactory(IAsyncResult result)
    {
        factory.EndOpen(result);
 
        channel = factory.CreateChannel(new EndpointAddress(serviceUrl));
 
        var openResult = channel.BeginOpen(r =>
        {
            if(!r.CompletedSynchronously)
                CompleteOpenChannel(r);
        }, channel);
 
        if(openResult.CompletedSynchronously)
            CompleteOpenChannel(openResult);
    }
 
    private void CompleteOpenChannel(IAsyncResult result)
    {
        channel.EndOpen(result);
 
        ReceiveMessage(channel);
    }
 
    private void ReceiveMessage(IInputChannel receivingChannel)
    {
        var result = receivingChannel.BeginReceive(r =>
        {
            if(!r.CompletedSynchronously)
                CompleteReceiveMessage(r);
        }, receivingChannel);
 
        if(result.CompletedSynchronously)
            CompleteReceiveMessage(result);
    }
 
    private void CompleteReceiveMessage(IAsyncResult result)
    {
        var message = channel.EndReceive(result);
 
        if(message == null)
            return;
 
        var action = message.Headers.GetHeader<string>("Action", "");
 
        if(!String.IsNullOrEmpty(action) && handlers.ContainsKey(action))
        {
            var handler = handlers[action];
            var serializer = new DataContractSerializer(handler.BodyType);
            var body = serializer.ReadObject(message.GetReaderAtBodyContents());
 
            uiThread.Post(p => handler.Invoke(body), null);
        }
 
        uiThread.Post(p => OnMessegeReceived(new MessageReceivedEventArgs(message)), null);
 
        ReceiveMessage(channel);
    }
 
    public void RegisterMessageHandler<T>(string action, Action<T> handler)
    {
        if(handlers.ContainsKey(action))
            throw new InvalidOperationException("Handler already registered for action " + action);
 
        handlers.Add(action, new MessageHandler<T>(handler));
    }
 
    public void Close()
    {
        var result = channel.BeginClose(r =>
        {
            if(!r.CompletedSynchronously)
                CompleteClose(r);
        }, null);
 
        if(result.CompletedSynchronously)
            CompleteClose(result);
    }
 
    private void CompleteClose(IAsyncResult result)
    {
        channel.EndClose(result);
    }
}
```

One of the downsides to the duplex clients
like this is that there&#39;s a single entry point for all messages being
received by the duplex client. We need a nice way to inspect the
incoming message and dispatch it to the appropriate handler. WCF has a
method for this on the server side, Messages have an Action, this
Action is used at the Endpoint to dispatch the message to the
appropriate method on the service. For instance the Action
&quot;CompiledExperience/IOrderService/Process&quot; is dispatched to the Process
method of IOrderService in the CompiledExperience namespace (not the C#
namespace but the namespace defined on the ServiceContract).

``` csharp
public class EncoderClient
{
    private readonly DuplexServiceClient serviceClient;
 
    public EncoderClient(string url)
    {
        serviceClient = new DuplexServiceClient(url);
        serviceClient.RegisterMessageHandler<EncodeSuccessMessage>("CompiledExperience/Video/Success", OnEncodeSuccess);
        serviceClient.RegisterMessageHandler<EncodeErrorMessage>("CompiledExperience/Video/Error", OnEncodeError);
        serviceClient.RegisterMessageHandler<EncodeProgressMessage>("CompiledExperience/Video/Progress", OnEncodeProgress);
    }
 
    public void Encode(int clientId, string fileName, string description, Size size)
    {
        serviceClient.Open();
 
        serviceClient.SendMessage("CompiledExperience/Video/Encode", new EncodeVideoMessage
           {
               ClientId = clientId,
               FileName = fileName,
               Description = description,
               Width = (int)size.Width,
               Height = (int)size.Height
           });
    }
 
    private void OnEncodeError(EncodeErrorMessage message)
    {
        serviceClient.Close();
    }
 
    private void OnEncodeSuccess(EncodeSuccessMessage message)
    {
        serviceClient.Close();
    }
 
    private void OnEncodeProgress(EncodeProgressMessage message)
    {
        // Report Progress here ...
    }
}
```

Sadly
the Messages received on the duplex client channel don&#39;t preserve their
Action. But we can duplicate something similar using the
Message.Headers collection. On the server we&#39;ll need to add a Header to
the Message named Action, it can be any string you want and doesn&#39;t
need to follow the same style pattern that the WCF Endpoint dispatcher
expects. This can be done like follows. 

``` csharp
public static class MessageFactory
{
    public static Message Create<T>(string action, T data)
    {
        var message = Message.CreateMessage(MessageVersion.Soap11, action, data, new DataContractSerializer(typeof(T)));
 
        message.Headers.Add(MessageHeader.CreateHeader("Action", "", action));
 
        return message;
    }
}
```

On the client side we want to be able to register a
handler for an action, as bonus we&#39;ll define what object the
Message.Body is a serialization of. The handler can take a reference to
that object. This way each handler is not dependent upon the Message
class at all. This part does depend on the serialization being the same on both ends so I&#39;ve gone with the default DataContract Serialization.

Hope this helps someone. 


