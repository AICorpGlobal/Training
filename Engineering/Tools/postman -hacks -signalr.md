Connecting to Asp.Net Core SignalR Hub from Postman with an example.
==================================================================== 

SignalR is an ideal solution for Asp.Net developers that makes it incredibly simple to deliver real-time auto page refresh solutions. 

For adding any sort of real-time web functionality to the Asp.Net application where the requirement is to the show user an update from the server without the user’s interaction on the browser, SignalR can be used.

A very common example is any modern chat application or a stock exchange application.

Unlike today’s common web applications which work on request-response patterns, SignalR supports server-push functionality where the server code can call the client’s code based on our requirement but not necessarily based on the request from the client application.

For this SignalR in Asp.Net uses WebSocket transport as its primary protocol. The [official Microsoft documentation](https://learn.microsoft.com/en-us/aspnet/signalr/overview/getting-started/introduction-to-signalr) has a detailed explanation of the SignalR technology.

Connecting to SignalR Hub during development is difficult because it works differently than the regular REST APIs.

After a few hours of research, I was able to call the hub from Postman by setting up a WebSocket request.

The primary goal of this article is to connect SignalR from Postman using WebSocket requests and send/receive the message.

Install and Configure SignalR In Asp.Net Core
=============================================

Install the SignalR client of the Asp.Net Core Project. I am using Asp.Net Core 7.0.

```
<PackageReference Include="Microsoft.AspNetCore.SignalR.Client" Version="7.0.10" />
```

Now, we will be creating a ChatHub class inheriting from the **Hub** provided by SignalR.

The **sendMessage**, method will be called by the client along with the content parameter and the **ReceiveMessage** method will be defined by the client.

The method below will send a message to all the connected clients. There are a few other predefined methods on SignalR for sending user or user-group-specific messages.

```
public sealed class ChatHub : Hub
{
    public async Task SendMessage(string content)
    {
        await Clients.All.SendAsync("ReceiveMessage", content);
    }
}
```

Lastly, you need to register the SignalR service and define the route pattern to the ChatHub class.

```
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSignalR();
var app = builder.Build();
app.MapHub<ChatHub>("chat-hub");
app.Run();
```

Here **chat-hub** will be the part of url that we will be using while calling ChatHub from Postman later.

Connect SignalR Hub using Postman
=================================

Create a new WebSocket request in Postman.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*xHXcFIRD3GcQ2TDzCy1MZA.png)

I assume you are using localhost as a development URL So, make sure the project is running on your localhost.

Enter the WebSocket URL without **_http://_** or **_https://_** and enter connect.

> If your SignalR project is already deployed, start your URL with **_ws://_** or **_wss://_**

```
localhost:5079/chat-hub
```

Once connected, enter the message in the following format and send the request.

```
{
  "protocol": "json",
  "version": 1
}
```

The message above is telling the signalR server about the communication message format.

**IMPORTANT**: The message needs to have **0x1E** characters at the end which seems kind of weird to me. But it is how it works.

I used the [insert-unicode](https://marketplace.visualstudio.com/items?itemName=brunnerh.insert-unicode) extension on VS Code and copied it from VSCode to Postman.

My Postman looks like this once the request succeeds and I get the response from SignalR Hub.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*F8zTeaBQpAhwvyCj0UUwWQ.png)

We have not sent the actual request yet. So far we have SignalR hub connected with Postman WebSocket and it’s ready to accept the the actual request.

Let’s do it with the actual request.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*h6c2bmR8MOl3Kk44mSV3RA.png)

I have been able to get the response back from Hub. The important thing to notice here is that the response has the target as **ReceiveMessage** which denotes the function that we have mentioned on ChatHub.

ReceiveMessage is the function that we will be creating on the Client and updating our UI based on the response data.
