# Client protocol description

This chapter aims to help developers to implement new client library or understand
how already implemented clients work. This chapter is not complete. I will
update it from time to time with additional information.

Centrifugo already has Javascript, Go, iOS, Android clients to connect your application
users.

One of the ways to understand how to implement new client is looking at source code
of [centrifuge-js](https://github.com/centrifugal/centrifuge-js/blob/master/src/centrifuge.js) or [centrifuge-go](https://github.com/centrifugal/centrifuge-go/blob/master/centrifuge.go).

Currently websocket is the only available transport to implement client. Centrifugo also support
SockJS connections from browser but it's only for browser usage, there is no reason to use it
from other environments. All communication done via exchanging JSON messages.

Let's look at client protocol step-by-step.

Connect, subscribe on channel and wait for published messages
-------------------------------------------------------------

Websocket endpoint is:

```
ws://your_centrifugo_server.com/connection/websocket
```

Or in case of using TLS:

```
wss://your_centrifugo_server.com/connection/websocket
```

What client should do first is to create Websocket connection to this endpoint.

After successful connection client must send `connect` command to server to authorize itself.

`connect` command is a JSON structure like this (all our examples here use Javascript
language, but the same can be applied to any language):

```javascript
var message = {
    "uid": "UNIQUE COMMAND ID",
    "method": "connect",
    "params": {
        "user": "USER ID STRING",
        "timestamp": "STRING WITH CURRENT TIMESTAMP SECONDS",
        "info": "OPTIONAL JSON ENCODED STRING",
        "token": "SHA-256 HMAC TOKEN GENERATED FROM PARAMETERS ABOVE"
    }
}

connection.send(JSON.stringify(message))
```

Look at `method` key with a name of our command – `connect`.

Centrifugo can parse an array of messages in one request, so you can add command above into
array and send result to Centrifugo server over Websocket connection established before:

```javascript
var messages = [message]
connection.send(JSON.stringify(messages))
```

Description of `connect` command parameters described in a chapter about javascript client.

In short here:

* `user` - current application user ID (string)
* `timestamp` - current Unix timestamp as seconds (string)
* `info` - optional JSON string with client additional information (string)
* `token` - SHA-256 HMAC token generated on backend (based on secret key from
    Centrifugo configuration) to sign parameters above.

Application backend must provide all these connection parameters (together with generated
HMAC SHA-256 token) to client (pass to template when client opens web page for example).

After receiving `connect` command over Websocket connection Centrifugo server uses the
same algorithm (HMAC SHA-256) to generate the token. Correct token proves that client
provided valid user ID, timestamp and info in its `connect` message.

*Note* that Centrifugo can also allow non-authenticated users to connect to it (for example
sites with public stream with notifications where all visitors can see new events in real-time
without actually logging in). For this case backend must generate token using empty string as
user ID. In this scenario `anonymous` access must be enabled for channels explicitly in
configuration of Centrifugo.

What you should do next is wait for response from server to `connect` command you just sent.

In general structure that will come from Centrifugo server to your client looks like this:

```javascript
[{response}, {response}, {response}]
```

Or just single response

```
{response}
```

I.e. array of responses or one response to commands you sent before. I.e. in our case Centrifugo
will send to our client:

```javascript
[{connect_command_response}]
```

Or just:

```javascript
{connect_command_response}
```

So **client must be ready to process both arrays of responses and single object response. This rule
applies to all communication**.

Every `response` is a structure like this:

```javascript
{
    "uid": "ECHO BACK THE SAME UNIQUE COMMAND ID SENT IN REQUEST COMMAND",
    "method": "COMMAND NAME TO WHICH THIS RESPONSE REFERS TO",
    "error": "ERROR STRING, IF NOT EMPTY THEN SOMETHING WENT WRONG AND BODY SHOULD NOT BE PROCESSED",
    "body": "RESPONSE BODY, CONTAINS USEFUL RESPONSE DATA"
}
```

Javascript client uses `method` key to understand what to do with response. As Javascript is
evented IO language it just calls corresponding function to react on response. Unique `uid`
also can be used to implement proper responses handling in other languages. For example Go
client remember command `uid` to call some callback when it receives response from Centrifugo.

General rule - if response contains a non-empty `error` then server returned an error.

You should not get errors in normal workflow. If you get an error then most probably you
are doing something wrong and this must be fixed on development stages. It can also be
`internal server error` from Centrifugo. Only developers should see text of protocol errors
– they are not supposed to be shown to your application clients.

In case of successful `connect` response body is:

```javascript
{
    "client": "UNIQUE CLIENT ID SERVER GAVE TO THIS CONNECTION",
    "expires": "false",
    "expired": false,
    "ttl": 0
}
```

At moment let's just speak about `client` key. This is unique client ID Centrifugo set
to this connection.

As soon your client successfully connected and got its unique connection ID it is ready to
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

Just send this `subscribe` command in the same way as `connect` command before.

After you received successful response on this `subscribe` command your client will receive
messages published to this channel. Those messages will be delivered through Websocket
connection as response with method `message`. I.e. response will look like this:

```
{
    "method": "message",
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

`body` of `message` response contains `channel` to which message corresponds and `data`
key - this is an actual JSON that was published into that channel.

This is enough to start with - client established connection, authorized itself sending `connect`
command, subscribed on channel to receive new messages published into that channel. This is a core
Centrifugo functionality. There are lots of other things to cover – channel presence information,
channel history information, connection expiration, private channel subscriptions, join/leave events
and more but in most cases all you need from Centrifugo - subscribe on channels and receive new
messages from those channels as soon as your backend published them into Centrifugo server API.

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

Some of this methods used for client to server commands (`publish`, `presence`, `history` etc which
then get a response from server with the same `method` and unique `uid` in it), some for server to
clients (for example `join`, `leave`, `message` – which just come from server in any time when
corresponding event occurred).

We have already seen `connect`, `subscribe` and `publish` above. Let's describe remaining.

Client to server commands
-------------------------

`connect` - send authorization parameters to Centrifugo so your connection could start subscribing
on channels.

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'connect',
    'params': {
        'user': "USER ID STRING",
        'timestamp': "STRING WITH CURRENT TIMESTAMP SECONDS"
        'info': "OPTIONAL JSON ENCODED STRING",
        'token': "SHA-256 HMAC TOKEN GENERATED FROM PARAMETERS ABOVE"
    }
}
```

`subscribe` - allows to subscribe on channel after client successfully connected

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
        "channel": "CHANNEL TO UNSUBSCRIBE"
    }
}
```

`publish` - allows clients directly publish messages into channel (application backend code will never
know about this message). `publish` must be enabled for channel in sever configuration so this command
can work (otherwise Centrifugo will return `permission denied` error in response).

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

`presence` – allows to ask server for channel presence information (`presence` must be enabled for
channel in server configuration or Centrifugo will return `not available` error in response)

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "presence",
    "params": {
        "channel": "CHANNEL"
    }
}
```

`history` – allows to ask server for channel history information (history must be enabled for
channel in server configuration using `history_lifetime` and `history_size` options or Centrifugo
will return `not available` error in response)

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "history",
    "params": {
        "channel": "CHANNEL"
    }
}
```

`ping` - allows to send ping command to server, server will answer this command with `ping`
response.

```javascript
message = {
    'uid': 'UNIQUE COMMAND ID',
    "method": "ping"
}
```

Responses of client to server commands
======================================

As soon as your client sent command to server it should then receive a corresponding response.
Let's look at those response messages in detail.

TODO: write about responses

Server to client commands
=========================

`message` - new message published into channel current client subscribed to. Response
for message coming over connection looks like this:

```javascript
{
    "method":"message",
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

`join` - someone joined a channel current client subscribed to. Note that `join_leave` option must
be enabled for channel in server configuration to receive this type of messages. `body` of this message
contains information about new subscribed client.

```javascript
{
    "method":"join",
    "body": {
        "channel":"$public:chat",
        "data": {
            "user":"2694",
            "client":"3702659c-f28a-4166-5b44-115d9b544b29",
            "default_info": {
                "first_name":"Alexandr",
                "last_name":"Emelin"
            },
            "channel_info": {
                "channel_extra_info_example":"you can add additional JSON data when authorizing"
            }
        }
    }
}
```

`leave` - someone left channel current client subscribed to. Note that `join_leave` option must
be enabled for channel in server configuration to receive this type of messages. `body` of this message
contains information about unsubscribed client.

```javascript
{
    "method":"leave",
    "body": {
        "channel":"$public:chat",
        "data": {
            "user":"2694",
            "client":"3702659c-f28a-4166-5b44-115d9b544b29",
            "default_info": {
                "first_name":"Alexandr",
                "last_name":"Emelin"
            },
            "channel_info": {
                "channel_extra_info_example":"you can add additional JSON data when authorizing"
            }
        }
    }
}
```

### Private channel subscriptions.

As you could see successful connect response body has `client` field - a unique client connection issued
by Centrifugo to this particular client connection. It's important because it's used when obtaining
private channel sign.

We've already seen above that in general case (non-private channel subscription) subscription request
that must be sent by client to Centrifugo looks like this:

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'subscribe',
    'params': {
        'channel': "CHANNEL TO SUBSCRIBE"
    }
}
```

When subscribing on private channel client must also provide additional fields in `params` object:

```javascript
var message = {
    'uid': 'UNIQUE COMMAND ID',
    'method': 'subscribe',
    'params': {
        'channel': "channel to subscribe",
        'client': "current client ID",
        'info': "additional private channel JSON string info",
        'sign': "string channel sign generated on app backend based on client ID and optional info"
    }
}
```

See [chapter about signs](./tokens_and_signatures.md) to get more knowledge about how to generate such
private channel sign on your backend side. In case of Javascript client we send client ID with private
channel names to backend automatically in AJAX request so all that user need is to check user
permissions, generate valid private channel sign and return in response. In case of other clients there
is no convenient way (such as AJAX in web) to get data from backend - so it's up to library user to
decide how he wants to obtain channel sign. Client library should at least provide mechanism to give
library user client id of current connection and mechanism to set `client`, `info` and `sign` fields
of subscription request `params`.

To be continued...

