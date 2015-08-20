# Client protocol description

This chapter aims to help developers to implement new client library. This chapter is a
work in progress. I will update it from time to time with additional information.

Centrifugo already has Javascript client to connect your application users from
web browser. But Centrifugo Websocket client connection endpoint can be used to
connect clients from other environments. For example from mobile applications.
There are no other client libraries available at moment but the goal of this document
is to help implement new client library, for example in Java for Android or in
Objective-C/Swift for iOS applications.

One of the ways to understand how to implement new client is looking at source code
of centrifuge-js - javascript client.

But let's look at protocol step-by-step.

Connect, subscribe on channel and wait for published messages
-------------------------------------------------------------

Websocket endpoint is:

```
ws://your_centrifugo_server.example.com/connection/websocket
```

What client first should do is create Websocket connection to this endpoint.

After successful connection client must send `connect` command to server to authorize
itself. This is a structure like this:

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'connect',
    'params': {
        'user': "USER ID STRING",
        'project': "PROJECT KEY STRING",
        'timestamp': "STRING WITH CURRENT TIMESTAMP SECONDS"
        'info': "OPTIONAL JSON ENCODED STRING",
        'token': "SHA-256 HMAC TOKEN GENERATED FROM PARAMETERS ABOVE"
    }
}
```

Look at `method` key with a name of our command - `connect`.

Centrifugo allows to send an array of messages in one request, so you can add command above
into array and send result to Centrifugo server over Websocket connection established before:

```javascript
var messages = [message]
connection.send(JSON.stringify(messages))
```

What you should do next is wait for response from server to `connect` command you just sent.

In general structure that will come from Centrifugo server to your client looks like this:

```javascript
[response, response, response]
```

I.e. array of responses to commands you sent before. I.e. in our case Centrifugo will send to our client:

```javascript
[connect_command_response,]
```

I.e. an array with single response to `connect` command we sent.

In general single response structure looks like this:

```javascript
{
    "uid": "THE SAME UNIQUE COMMAND ID SENT IN REQUEST COMMAND",
    "method": "COMMAND NAME TO WHICH THIS RESPONSE REFERS TO",
    "error": "ERROR - IF NOT EMPTY THEN SOMETHING WENT WRONG AND BODY SHOULD NOT BE PROCESSED",
    "body": "RESPONSE BODY - IF NO ERROR CONTAINS USEFUL RESPONSE DATA"
}
```

Javascript client uses `method` key to understand what to do with response. As Javascript is
evented IO language it just calls corresponding function to react on response. Unique `uid` also
can be used to implement proper responses handling.

General rule - if response contains a non-empty `error` then server returned an error and client
should not process response `body` data.

In case of successful `connect` response body is:

```javascript
{
    "client": "UNIQUE CLIENT ID SERVER GAVE TO THIS CONNECTION",
    "expired": false,
    "ttl": null
}
```

At moment let's just speak about `client` key. This is unique client connection ID.

As soon your client successfully connected and got its connection ID it is ready to
subscribe on channels.

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'subscribe',
    'params': {
        'channel': "CHANNEL TO SUBSCRIBE"
    }
}
```

Send it in a same way as `connect` command before.

After you received successful response on this `subscribe` command your client will receive
messages published to this channel. Those messages will be delivered through Websocket connection
as general response and response method will be `message` in this case. I.e. response will look
like this:

```
{
    "method":"message",
    "error":null,
    "body": {
        "uid": "8d1f6279-2d13-45e2-542d-fac0e0f1f6e0",
        "timestamp":"1439715024",
        "info":{
            "user":"42",
            "client":"73cd5abb-03ed-40bc-5c87-ed35df732682",
            "default_info":null,
            "channel_info":null
        },
        "channel":"jsfiddle-chat",
        "data": {
            "input":"hello world"
        },
        "client":"73cd5abb-03ed-40bc-5c87-ed35df732682"
    }
}
```

`body` of `message` response contains `channel` to which message corresponds and `data` key - this
is an actual JSON that was published into that channel.

This is enough to start with - client established connection, authorized itself sending `connect`
command, subscribed on channel to receive new messages published into that channel. This is a core
Centrifugo functionality. There are lots of other things to cover - channel presence information,
channel history information, connection expiration, private channel subscriptions and more but in
most cases all you need from Centrifugo - subscribe on channels and receive new messages from those
channels.

Available methods
-----------------

Lets now look at all available methods your client can send or receive:

```
connect
disconnect
subscribe
unsubscribe
publish
presence
history
join
leave
message
refresh
ping
```

Some of this methods used for client to server commands (`publish`, `presense`, `history` etc which then get a response from server with the same `method` and unique `uid` in it), some for server to clients (for example `join`, `leave`, `message` – which just come from server in any time when corresponding event occurred).

We have already seen `connect`, `subscribe` and `publish` above. Let's describe remaining.

Client to server commands
-------------------------

`connect` - described above

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'connect',
    'params': {
        'user': "USER ID STRING",
        'project': "PROJECT KEY STRING",
        'timestamp': "STRING WITH CURRENT TIMESTAMP SECONDS"
        'info': "OPTIONAL JSON ENCODED STRING",
        'token': "SHA-256 HMAC TOKEN GENERATED FROM PARAMETERS ABOVE"
    }
}
```

`subscribe` - described above

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'subscribe',
    'params': {
        'channel': "CHANNEL TO SUBSCRIBE"
    }
}
```

`unsubscribe` - allows to unsubscribe from channel

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "unsubscribe",
    "params": {
        "channel": this.channel
    }
}
```

`publish` - allows clients directly publish messages into channel (application backend code will never know about this message). `publish` must be enabled for channel in sever configuration so this command can work.

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "publish",
    "params": {
        "channel": "CHANNEL",
        "data": {}  // JSON DATA TO PUBLISH
    }
}
```

`presence` – allows to ask server for channel presence information (`presence` must be enabled for channel in server configuration)

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "presence",
    "params": {
        "channel": "CHANNEL"
    }
}
```

`history` – allows to ask server for channel history information (history must be enabled for channel in server configuration using `history_lifetime` and `history_size` options)

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "history",
    "params": {
        "channel": "CHANNEL"
    }
}
```

`ping` - allows to send ping command to server, server should answer this command.

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "ping",
    "params": {}
}
```

To be continued...

