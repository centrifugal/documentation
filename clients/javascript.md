Javascript browser client
=========================

At this moment you know how Centrifugo server implemented and how it works. It's time to
connect your web application users to the server from web browser.

For this purpose javascript client with simple API exists.

* [Install and quick start](#install-and-quick-start)
* [Connection parameters](#connection-parameters)
* [Configuration parameters](#configuration-parameters)
* [Client API](#client-api)
* [Private channels](#private-channels)
* [Connection check](#connection-check)

The source code of javascript client located in [repo on Github](https://github.com/centrifugal/centrifuge-js).

Javascript client can connect to the server in two ways: using pure Websockets or using
[SockJS](https://github.com/sockjs/sockjs-client) library to be able to use various
available fallback transports if client browser does not support Websockets.

With javascript client you can:

* connect your user to real-time server
* subscribe on channel and listen to all new messages published into this channel
* get presence information for channel (all clients currently subscribed on channel)
* get history messages for channel
* receive join/leave events for channels (when someone subscribes on channel or
    unsubscribes from it)
* publish new messages into channels

*Note, that in order to use presence, history, join/leave and publish – corresponding options
must be enabled in Centrifugo channel configuration (on top level or for channel namespace).*

If you are searching for old API docs (`centrifuge-js` < 1.3.0) - [you can find it here](https://github.com/centrifugal/documentation/tree/c69ca51f21c028a6b9bd582afdbf0a5c13331957/client)


## Install and quick start

The simplest way to use javascript client is including it into your web page using `script` tag:

```html
<script src="centrifuge.js"></script>
```

Download client its [from repository](https://github.com/centrifugal/centrifuge-js).

Browser client is also available via `npm` and `bower`. So you can use:

```bash
npm install centrifuge
```

Or:

```bash
bower install centrifuge
```

If you want to use SockJS you must also import SockJS client before centrifuge.js

```html
<script src="//cdn.jsdelivr.net/sockjs/1.1/sockjs.min.js" type="text/javascript"></script>
<script src="centrifuge.js" type="text/javascript"></script>
```

**If you want to support Internet Explorer < 8** then you also need to include JSON
polyfill library:

```html
<script src="//cdnjs.cloudflare.com/ajax/libs/json3/3.3.2/json3.min.js" type="text/javascript"></script>
<script src="//cdn.jsdelivr.net/sockjs/1.1/sockjs.min.js" type="text/javascript"></script>
<script src="centrifuge.js" type="text/javascript"></script>
```

As soon as you included all libraries you can create new `Centrifuge` object instance,
subscribe on channel and call `.connect()` method to make actual connection to
Centrifugo server:

```javascript
<script type="text/javascript">

    var centrifuge = new Centrifuge({
        url: 'http://centrifuge.example.com/connection',
        user: "USER ID",
        timestamp: "UNIX TIMESTAMP SECONDS",
        token: "SHA-256 HMAC TOKEN"
    });

    centrifuge.subscribe("news", function(message) {
        console.log(message);
    });

    centrifuge.connect();

</script>
```

In example above we initialize `Centrifuge` object instance, subscribe on channel
`news`, print all new messages received from channel `news` into console and actually
make connection to Centrifugo. And that's all code which required for simple real-time
messaging handling on client side!

***`Centrifuge` object is an instance of [EventEmitter](https://github.com/Olical/EventEmitter/blob/master/docs/api.md).***

Parameters `url`, `user`, `timestamp` and `token` are required. Let's look at these
connection parameters and other configuration options in detail.

## Connection parameters

As we showed above to initialize `Centrifuge` object you must provide connection
parameters: `url`, `user`, `timestamp`, `token`, optional `info`.

**Note that all connection parameters (except url maybe) must come to your Javascript code from
your application backend**. You can render template with these connection parameters, or you can
pass them in cookie, or even make an AJAX GET request from your Javascript code to get  `user`,
`timestamp`, `info` and `token`.

Let's see for what each option is responsible for.

#### url (required)

`url` – is an endpoint of Centrifugo server to connect to.

If your Centrifugo server sits on domain `centrifugo.example.com` then:

* SockJS endpoint will be `http://centrifugo.example.com/connection`
* Pure Websocket endpoint will be `ws://centrifugo.example.com/connection/websocket`

Remember to include SockJS library on your page when you want to use SockJS.

If your Centrifugo works through SSL (**this is recommended btw**) then endpoint
addresses must start with `https` (SockJS) and `wss` (Websocket) instead of `http`
and `ws`.

You can also set `url` to just `http://centrifugo.example.com` and javascript client will
detect which endpoint to use (SockJS or Websocket) automatically based on SockJS library availability.

#### user (required)

`user` **string** is your web application's current user ID. **It can be empty if you don't
have logged in users but in this case you must enable** `anonymous` access option for
channels in Centrifugo configuration (setting `anonymous: true` on top level or for channel
namespace).

Note, that **it must be string type** even if your application uses numbers as user ID.
Just convert that user ID number to string.

#### timestamp (required)

`timestamp` string is UNIX server time in seconds when connection token (see below)
was generated.

Note, that most programming languages by default return UNIX timestamp as float value.
Or with microseconds included. Centrifugo server **expects only timestamp seconds
represented as string**. For example for Python to get timestamp in a correct format
use `"%.0f" % time.time()` (or just `str(int(time.time()))`) so the result be something
like `"1451991486"`.

#### token (required)

`token` is a digest string generated by your web application backend based on Centrifugo
`secret` key, `user` ID, `timestamp` (and optional `info` - see below).

To create token HMAC SHA-256 algorithm is used. To understand how to generate client
connection token see special chapter [Tokens and signatures](../server/tokens_and_signatures.md).

**For Python, Ruby, NodeJS, Go and PHP we already have functions to generate client
token in API libraries**.

Correct token guarantees that connection request to Centrifugo contains valid information
about user ID and timestamp. Token is similar to HTTP cookie, client must never show it
to anyone else. Also remember that you should consider *using private channels* when working
with confidential data.

#### info (optional)

You can optionally provide extra parameter `info` when connecting to Centrifugo, i.e.:

```javascript
var centrifuge = new Centrifuge({
    url: 'http://centrifuge.example.com/connection',
    user: 'USER ID',
    timestamp: 'UNIX TIMESTAMP',
    info: '{"first_name": "Alexandr", "last_name": "Emelin"}',
    token: 'TOKEN'
});
```

`info` is an additional information about connection. It must be **valid encoded JSON string**.
But to prevent client sending wrong `info` **this JSON string must be used while generating
token**.

If you don't want to use `info` - you can just omit this parameter while connecting to Centrifugo.
But if you omit it then make sure that info string have not been used in token generation
(i.e. `info` must be empty string).

## Configuration parameters

Let's also look at optional configuration parameters available when initializing
`Centrifuge` object instance.

#### transports

In case of using SockJS additional configuration parameter can be used - `transports`.

It defines allowed SockJS transports and by default equals

```javascript
var centrifuge = new Centrifuge({
    ...
    transports: [
        'websocket', 'xdr-streaming', 'xhr-streaming',
        'eventsource', 'iframe-eventsource', 'iframe-htmlfile',
        'xdr-polling', 'xhr-polling', 'iframe-xhr-polling', 'jsonp-polling'
    ]
});
```

i.e. all possible SockJS transports.

So to say `centrifuge-js` to use only `websocket` and `xhr-streaming` transports when
using SockJS endpoint:

```javascript
var centrifuge = new Centrifuge({
    url: 'http://centrifuge.example.com/connection',
    user: 'USER ID',
    timestamp: 'UNIX TIMESTAMP',
    info: '{"first_name": "Alexandr", "last_name": "Emelin"}',
    token: 'TOKEN',
    transports: ["websocket", "xhr-streaming"]
});
```

#### sockJS

**new in 1.3.7**. `sockJS` option allows to explicitly provide SockJS client object to Centrifuge client.

For example this can be useful if you develop in ES6 using imports.

```javascript
import Centrifuge from 'centrifuge'
import SockJS from 'sockjs-client'

var centrifuge = new Centrifuge({
    url: 'http://centrifuge.example.com/connection',
    user: 'USER ID',
    timestamp: 'UNIX TIMESTAMP',
    info: '{"first_name": "Alexandr", "last_name": "Emelin"}',
    token: 'TOKEN',
    sockJS: SockJS
});
```

#### debug

`debug` is a boolean option which is `false` by default. When enabled lots of various debug
messages will be logged into javascript console. Mostly useful for development or
troubleshooting.

#### insecure

`insecure` is a boolean option which is `false` by default. When enabled client will connect
to server in insecure mode - read about this mode in [special docs chapter](../mixed/insecure_modes.md).

This option nice if you want to use Centrifugo for quick real-time ideas prototyping, demos as
it allows to connect to Centrifugo without `token`, `timestamp` and `user`. And moreover without
application backend! Please, [read separate chapter about insecure modes](../mixed/insecure_modes.md).

#### retry

When client disconnected from server it will automatically try to reconnect using exponential
backoff algorithm to get interval between reconnect attempts which value grows exponentially.
`retry` option sets minimal interval value in milliseconds. Default is `1000` milliseconds.

#### maxRetry

`maxRetry` sets upper interval value limit when reconnecting. Or your clients will never reconnect
as exponent grows very fast:) Default is `20000` milliseconds.

#### resubscribe

`resubscribe` is boolean option that allows to disable automatic resubscribing on
subscriptions. By default it's `true` - i.e. you don't need to manually handle
subscriptions resubscribing and no need to wait `connect` event triggered (first
time or when reconnecting) to start subscribing. `centrifuge-js` will by default
resubscribe automatically when connection established.

#### server

`server` is SockJS specific option to set server name into connection urls instead
of random chars. See SockJS docs for more info.

#### authEndpoint

`authEndpoint` is url to use when sending auth request for authorizing subscription
on private channel. By default `/centrifuge/auth/`. See also useful related options:

* `authHeaders` - map of headers to send with auth request (default `{}``)
* `authParams` - map of params to include in auth url (default `{}`)
* `authTransport` - transport to use for auth request (default `ajax`, possible value `jsonp`)

#### refreshEndpoint

`refreshEndpoint` is url to use when refreshing client connection parameters when
connection check mechanism enabled in Centrifugo configuration. See also related
options:

* `refreshHeaders` - map of headers to send with refresh request (default `{}``)
* `refreshParams` - map of params to include in refresh url (default `{}`)
* `refreshTransport` - transport to use for refresh request (default `ajax`, possible value `jsonp`)
* `refreshData` - send extra data in body (as JSON payload) when sending AJAX POST refresh request.
* `refreshAttempts` - limit amount of refresh requests before giving up (by default `null` - unlimited)
* `refreshFailed` - callback function called when `refreshAttempts` came to the end. By default `null` - i.e. nothing called.

## Client API

When `Centrifuge` object properly initialized then it is ready to start communicating
with server.

#### connect method

As we showed before, we must call `connect()` method to make an actual connection
request to Centrifugo server:

```javascript
var centrifuge = new Centrifuge({
    // ...
});

centrifuge.connect();
```

`connect()` calls actual connection request to server with connection parameters and
configuration options you provided during initialization.

#### connect event

After connection will be established and client credentials you provided authorized
then `connect` event on `Centrifuge` object instance will be called.

You can listen to this setting event listener function on `connect` event:

```javascript
centrifuge.on('connect', function(context) {
    // now client connected to Centrifugo and authorized
});
```

What's in `context`:

```javascript
{
    client: "79ec54fa-8348-4671-650b-d299c193a8a3",
    transport: "raw-websocket",
    latency: 21
}
```

* `client` – client ID Centrifugo gave to this connection (string)
* `transport` – name of transport used to establish connection with server (string)
* `latency` – latency in milliseconds (int). This measures time passed between sending
    `connect` client protocol command and receiving connect response. New in 1.3.1

#### disconnect event

`disconnect` event fired on centrifuge object every time client disconnects for
some reason. This can be network disconnect or disconnect initiated by Centrifugo server.

```javascript
centrifuge.on('disconnect', function(context) {
    // do whatever you need in case of disconnect from server
});
```

What's in `context`?

```javascript
{
    reason: "connection closed",
    reconnect: true
}
```

* `reason` – the reason of client's disconnect (string)
* `reconnect` – flag indicating if client will reconnect or not (boolean)


#### error event

`error` event called every time on centrifuge object when response with error received.
In normal workflow it will never be happen. But it's better to log these errors to detect
where problem with connection is.

This event handler is a general error messages sink - it will receives all messages received
from Centrifugo containing error so it could also receive message resulting in `error` event
for subscription (see below). The difference is that this event handler exists mostly for
logging purposes to help developer fix possible problems - while other errors (subscription
error or `publish`, `presence`, `history` call errors) can be theoretically handled to retry
call or resubscribe maybe.

```javascript
centrifuge.on('error', function(error) {
    // handle error in a way you want, here we just log it into browser console.
    console.log(error)
});
```

What's in `error`?

```javascript
{
    "message": {
        "method": "METHOD",
        "error": "ERROR DESCRIPTION",
        "advice": "OPTIONAL ERROR ADVICE"
    }
}
```

`message` – message from server containing error. It's a raw protocol message resulted in
error event because it contains `error` field. At bare minimum it's recommended to log these
errors. In normal workflow such errors should never exist and must be a signal for developer
that something goes wrong.


#### disconnect method

In some cases you may need to disconnect your client from server, use `disconnect` method to
do this:

```javascript
centrifuge.disconnect();
```

After calling this client will not try to reestablish connection periodically. You must call
`connect` method manually again.


## Subscriptions

Of course being just connected is useless. What we usually want from Centrifugo is to
receive new messages published into channels. So our next step is `subscribe` on channel
from which we want to receive real-time messages.


### subscribe method

To subscribe on channel we must use `subscribe` method of `Centrifuge` object instance.

The simplest usage that allow to subscribe on channel and listen to new messages is:

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle new message coming from channel "news"
    console.log(message);
});
```

And that's all! For lots of cases it's enough! But let's look at possible events that
can happen with subscription:

* `message` – called when new message received (callback function in our previous example is `message`
    event callback btw)
* `join` – called when someone joined channel
* `leave` – called when someone left channel
* `subscribe` – called when subscription on channel successful and acknowledged by Centrifugo
    server. It can be called several times during javascript code lifetime as browser client
    automatically resubscribes on channels after successful reconnect (caused by temporary
    network disconnect for example or Centrifugo server restart).
* `error` – called when subscription on channel failed with error. It can called several times
    during javascript code lifetime as browser client automatically resubscribes on channels
    after successful reconnect (caused by temporary network disconnect for example or Centrifugo
    server restart).
* `unsubscribe` – called every time subscription that was successfully subscribed
    unsubscribes from channel (can be caused by network disconnect or by calling
    `unsubscribe` method of subscription object)

Don't be frightened by amount of events available. In most cases you only need some of them
until you need full control to what happens with your subscriptions. We will look at format
of messages for this event callbacks later below.

There are 2 ways setting callback functions for events above.

First is providing object containing event callbacks as second argument to `subscribe` method.

```javascript
var callbacks = {
    "message": function(message) {
        // See below description of message format
        console.log(message);
    },
    "join": function(message) {
        // See below description of join message format
        console.log(message);
    },
    "leave": function(message) {
        // See below description of leave message format
        console.log(message);
    },
    "subscribe": function(context) {
        // See below description of subscribe callback context format
        console.log(context);
    },
    "error": function(errContext) {
        // See below description of subscribe error callback context format
        console.log(err);
    },
    "unsubscribe": function(context) {
        // See below description of unsubscribe event callback context format
        console.log(context);
    }
}

var subscription = centrifuge.subscribe("news", callbacks);
```

Another way is setting callbacks using `on` method of subscription. Subscription object
is event emitter so you can simply do the following:

```javascript
var subscription = centrifuge.subscribe("news");

subscription.on("message", messageHandlerFunction);
subscription.on("subscribe", subscribeHandlerFunction);
subscription.on("error", subscribeErrorHandlerFunction);
```

***Subscription objects are instances of [EventEmitter](https://github.com/Olical/EventEmitter/blob/master/docs/api.md).***


### join and leave events of subscription

As you know you can enable `join_leave` option for channel in Centrifugo configuration.
This gives you an opportunity to listen to `join` and `leave` events in those channels.
Just set event handlers on `join` and `leave` events of subscription.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
}).on("join", function(message) {
    console.log("Client joined channel");
}).on("leave", function(message) {
    console.log("Client left channel");
});
```


### subscription event context formats

We already know how to listen for events on subscription. Let's look at format of
messages event callback functions receive as arguments.

#### format of message event context

Let's look at message format of new message received from channel:

```javascript
{
    "uid":"6778c79f-ccb2-4a1b-5768-2e7381bc5410",
    "channel":"$public:chat",
    "data":{"input":"hello"},
}
```

I.e. `data` field contains actual data that was published.

Message can optionally contain `client` field (client ID that published message) - if
it was provided when publishing new message:

```javascript
{
    "uid":"6778c79f-ccb2-4a1b-5768-2e7381bc5410",
    "channel":"$public:chat",
    "data":{"input":"hello"},
    "client":"7080fd2a-bd69-4f1f-6648-5f3ceba4b643"
}
```

And it can optionally contain additional client `info` in case when this message was
published by javascript client directly using `publish` method (see details below):

```javascript
{
    "uid":"6778c79f-ccb2-4a1b-5768-2e7381bc5410",
    "info":{
        "user":"2694",
        "client":"7080fd2a-bd69-4f1f-6648-5f3ceba4b643",
        "default_info":{"name":"Alexandr"},
        "channel_info":{"extra":"extra JSON data when authorizing private channel"}
    },
    "channel":"$public:chat",
    "data":{"input":"hello"},
    "client":"7080fd2a-bd69-4f1f-6648-5f3ceba4b643"
}
```


#### format of join/leave event message

I.e. `on("join", function(message) {...})` or `on("leave", function(message) {...})`

```javascript
{
    "channel":"$public:chat",
    "data":{
        "user":"2694",
        "client":"2724adea-6e9b-460b-4430-a9f999e94c36",
        "default_info":{"first_name":"Alexandr"},
        "channel_info":{"extra":"extra JSON data when authorizing"}
    }
}
```

`default_info` and `channel_info` exist in message only if not empty.


#### format of subscribe event context

I.e. `on("subscribe", function(context) {...})`

```javascript
{
    "channel": "$public:chat",
    "isResubscribe": true
}
```

`isResubscribe` – flag showing if this was initial subscribe (`false`) or resubscribe (`true`)


#### format of subscription error event context

I.e. `on("error", function(err) {...})`

```javascript
{
    "error": "permission denied",
    "advice": "fix",
    "channel": "$public:chat",
    "isResubscribe": true
}
```

`error` - error description
`advice` - optional advice (`retry` or `fix` at moment)
`isResubscribe` – flag showing if this was initial subscribe (`false`) or resubscribe (`true`)


#### format of unsubscribe event context

I.e `on("unsubscribe", function(context) {...})`

```
{
    "channel": "$public:chat"
}
```

I.e. it contains only `channel` at moment.


### presence method of subscription

`presence` allows to get information about clients which are subscribed on channel at
this moment. Note that this information is only available if `presence` option enabled
in Centrifugo configuration for all channels or for channel namespace.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.presence().then(function(message) {
    // presence data received
}, function(err) {
    // presence call failed with error
});
```

`presence` is internally a promise that will be resolved with data or error only
when subscription actually subscribed.

Format of success callback `message`:

```javascript
{
    "channel":"$public:chat",
    "data":{
        "2724adea-6e9b-460b-4430-a9f999e94c36": {
            "user":"2694",
            "client":"2724adea-6e9b-460b-4430-a9f999e94c36",
            "default_info":{"first_name":"Alexandr"}
        },
        "d274505c-ce63-4e24-77cf-971fd8a59f00":{
            "user":"2694",
            "client":"d274505c-ce63-4e24-77cf-971fd8a59f00",
            "default_info":{"first_name":"Alexandr"}
        }
    }
}
```

As you can see presence data is a map where keys are client IDs and values are objects
with client information.

Format of `err` in error callback:

```javascript
{
    "error": "timeout",
    "advice": "retry"
}
```

* `error` – error description (string)
* `advice` – error advice (string, "fix" or "retry" at moment)


### history method of subscription

`history` method allows to get last messages published into channel. Note that history
for channel must be configured in Centrifugo to be available for `history` calls from
client.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.history().then(function(message) {
        // history messages received
    }, function(err) {
        // history call failed with error
    });
});
```

Success callback `message` format:

```javascript
{
    "channel": "$public:chat",
    "data": [
        {
            "uid": "87219102-a31d-44ed-489d-52b1a7fa520c",
            "channel": "$public:chat",
            "data": {"input": "hello2"}
        },
        {
            "uid": "71617557-7466-4cbb-760e-639042a5cade",
            "channel": "$public:chat",
            "data": {"input": "hello1"}
        }
    ]
}
```

Where `data` is an array of messages published into channel.

Note that also additional fields can be included in messages - `client`, `info` if those
fields were in original messages.

`err` format – the same as for `presence` method.


### publish method of subscription

`publish` method of subscription object allows to publish data into channel directly
from client. The main idea of Centrifugo is server side only push. Usually your application
backend receives new event (for example new comment created, someone clicked like button
etc) and then backend posts that event into Centrifugo over API. But in some cases you may
need to allow clients to publish data into channels themselves. This can be used for demo
projects, when prototyping ideas for example, for personal usage. And this allow to make
something with real-time features without any application backend at all. Just javascript
code and Centrifugo.

**So to emphasize: using client publish is not an idiomatic Centrifugo usage. It's not for
production applications but in some cases (demos, personal usage, Centrifugo as backend
microservice) can be justified and convenient. In most real-life apps you need to send new
data to your application backend first (using the convenient way, for example AJAX request
in web app) and then publish data to Centrifugo over Centrifugo API.**

To do this you can use `publish` method. Note that just like presence and history publish
must be allowed in Centrifugo configuration for all channels or for channel namespace. When
using `publish` data will go through Centrifugo to all clients in channel. Your application
backend won't receive this message.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.publish({"input": "hello world"}).then(function() {
        // success ack from Centrifugo received
    }, function(err) {
        // publish call failed with error
    });
});
```

`err` format – the same as for `presence` method.


### unsubscribe method of subscription

You can call `unsubscribe` method to unsubscribe from subscription:

```javascript
subscription.unsubscribe();
```

### ready method of subscription

A small drawback of setting event handlers on subscription using `on` method is that event
handlers can be set after `subscribe` event of underlying subscription already fired. This
is not a problem in general but can be actual if you use one subscription (i.e. subscription
to the same channel) from different parts of your javascript application - so be careful.

For this case one extra helper method `.ready(callback, errback)` exists. This method calls
`callback` if subscription already subscribed and calls `errback` if subscription already
failed to subscribe with some error (because you subscribed on this channel before). So
when you want to call subscribe on channel already subscribed before you may find `ready()`
method useful:

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message;
});

// artificially model subscription to the same channel that happen after
// first subscription successfully subscribed - subscribe on the same
// channel after 5 seconds.
setTimeout(function() {
    var anotherSubscription = centrifuge.subscribe("news", function(message) {
        // another listener of channel "news"
    }).on("subscribe", function() {
        // won't be called on first subscribe because subscription already subscribed!
        // but will be called every time automatic resubscribe after network disconnect
        // happens
    });
    // one of subscribeSuccessHandler (or subscribeErrorHandler) will be called
    // only if subscription already subscribed (or subscribe request already failed).
    anotherSubscription.ready(subscribeSuccessHandler, subscribeErrorHandler);
}, 5000);
```

When called `callback` and `errback` of `ready` method receive the same arguments as
callback functions for `subscribe` and `error` events of subscription.


### Message batching

There is also message batching support. It allows to send several messages to server
in one request - this can be especially useful when connection established via one of
SockJS polling transports.

You can start collecting messages to send calling `startBatching()` method:

```javascript
centrifuge.startBatching();
```

When you want to actually send all collected messages to server call `flush()` method:

```javascript
centrifuge.flush();
```

Finally if you don't want batching anymore call `stopBatching()` method:

```javascript
centrifuge.stopBatching();
```

Call `stopBatching(true)` to flush all messages and stop batching:

```javascript
centrifuge.stopBatching(true);
```


## Private channels

If channel name starts with `$` then subscription on this channel will be checked via
AJAX POST request from javascript client to your web application backend.

You can subscribe on private channel as usual:

```javascript
centrifuge.subscribe('$private', function(message) {
    // process message
});
```

But in this case client will first check subscription via your backend sending POST request
to `/centrifuge/auth/` endpoint (by default, can be changed via configuration option
`authEndpoint`). This request will contain `client` parameter which is your connection
client ID and `channels` parameter - one or multiple private channels client wants to
subscribe to. Your server should validate all this subscriptions and return properly
signed responses.

There are also two public API methods which can help to subscribe to many private
channels sending only one POST request to your web application backend: `startAuthBatching`
and `stopAuthBatching`. When you `startAuthBatching` javascript client will collect
private subscriptions until `stopAuthBatching()` called – and then send them all at
once.

Read more about private channels in [special documentation chapter](../mixed/private_channels.md).


## Connection check

Javascript client has support for refreshing connection when `connection_lifetime` option
set in Centrifugo. See more details in [dedicated chapter](../server/connection_check.md).
