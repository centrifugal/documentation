# Client API methods

When centrifuge object properly initialized then it is ready to start communicating with server.

### connect

As we said before, we should actually call `connect()` method to make a connection to server:

```javascript
var centrifuge = new Centrifuge({
    url: "CONNECTION ENDPOINT",
    user: "USER ID",
    timestamp: "UNIX TIMESTAMP SECONDS",
    info: "OPTIONAL CONNECTION INFO AS JSON ENCODED STRING",
    token: "SHA256 HMAC TOKEN"
});


centrifuge.connect();
```

`connect()` calls actual connection request to server with connection parameters you provided
during initialization.

After connection will be established and client credentials you provided in initialization
authorized `connect` event on `centrifuge` object will be called.

You can listen it setting event listener on `connect` event:

```javascript
centrifuge.on('connect', function() {
    // now client connected to Centrifugo and authorized
});
```

### subscribe

Of course being just connected is useless. What we usually want from Centrifugo is to
receive new messages published into channels. So our next step is `subscribe` on channel
from which we want to receive messages.

To subscribe we must use `.subscribe(channel, callbacks)` method of `centrifuge` object.

The simplest usage that allow to subscribe on channel and listen to new messages is:

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle new message coming from channel "news"
    console.log(message);
});
```

For lots of cases it's enough. But let's look at possible events that can happen
with subscription:

* `message` – new message received (callback in our previous example is `message` event
    callback btw)
* `join` – someone joined channel
* `leave` – someone left channel
* `subscribe` – subscription on channel successful and acknowledged by Centrifugo server
* `error` – subscription on channel failed with error
* `subscribe:success` – called once when first attempt to subscribe finished with success
* `subscribe:error` – called once when first attempt to subscribe failed with error
* `resubscribe:success` – called every time resubscribing on subscription after
    disconnects succeeds.
* `resubscribe:error` – called every time resubscribing on subscription after
    disconnects failes with error.
* `unsubscribe` – called every time subscription that was successfully subscribed
    unsubscribes from channel (can be caused by network disconnect or by calling `unsubscribe` method of subscription object)

Don't be frightened by amount of events available. In most cases you only need some of them until
you need full control to what happens with your subscriptions.

There are 2 ways setting callback functions for events above.

First is providing object containing event callbacks as second argument to `subscribe` method.

```javascript
var callbacks = {
    "message": function(message) {
        console.log(message);
    },
    "join": function(message) {
        console.log(info);
    },
    "leave": function(message) {
        console.log(info);
    },
    "subscribe": function(subscription) {
        console.log(subscription);
    },
    "error": function(err) {
        console.log(err);
    }
}

var subscription = centrifuge.subscribe("news", callbacks);
```

Another way is setting callbacks using `on` method of subscription. Subscription object
is event emitter so you can simply do the following:

```
var subscription = centrifuge.subscribe("news");

subscription.on("message", messageHandlerFunction);
subscription.on("subscribe", subscribeHandlerFunction);
subscription.on("error", errorHandlerFunction);
```

The drawback of last approach is that event handlers can be set after `subscribe` event of
subscription already fired. This is not a problem in general but can be actual if you use
one subscription (i.e. subscription to the same channel) from different parts of your javascript
application. For this case there is one extra method `.ready()` that return promise object. This
promise resolves after first subscribe attempt with success or with error. So when you want to
call subscribe on channel already subscribed before you may find this `ready()` method useful:

```
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message;
});

// artificially model subscription to the same channel that happen after
// first subscription successfully subscribed
setTimeout(function() {
    var anotherSubscription = centrifuge.subscribe("news", function(message) {
        // another listener of channel "news"
    }).on("subscribe", function() {
        // won't be called because subscription already subscribed!
    });
    anotherSubscription.ready().then(subscribeSuccessHandler, subscribeErrorHandler);
}, 5000);
```

For this tricky situations events like `subscribe:success`, `subscribe:error`,
`resubscribe:success`, `resubscribe:error` exist.

As soon as subscription successfully subscribed (i.e. `subscribe` event happened) it's
possible to call `presence`, `history` and `publish` methods.

### presence

`presence` allows to get information about clients which are subscribed on channel at
this moment. Note that this information is only available if `presence` option enabled
in Centrifugo configuration for all channels or for namespace.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.on("subscribe", function(sub) {
    sub.presence(function(data) {
        // presence data received
    }, function(err) {
        // presence call failed with error
    });
});
```

### history

`history` method allows to get last messages published into channel. Note that history
for channel must be configured in Centrifugo to be available for `history` calls from
client.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.on("subscribe", function(sub) {
    sub.history(function(data) {
        // history messages received
    }, function(err) {
        // history call failed with error
    });
});
```

### publish

`publish` method of subscription object allows to publish data into channel directly
from client. The main idea of Centrifugo is server side only push. Usually your application
backend receives new event (for example new comment created, someone clicked like button
etc) and then backend posts that event into Centrifugo over API. But in some cases you may
need to allow clients to publish data into channels themselves. This can be used for demo
projects, when prototyping ideas for example, for personal usage. And this allow to make
something with real-time features without any application backend at all. Just javascript
code and Centrifugo.

To do this you can use `publish` method. Note that just like presence and history publish
must be allowed in Centrifugo configuration for all channels or for namespace. When using
`publish` data will go through Centrifugo to all clients in channel. Your application
backend won't receive this message.

```javascript
var subscription = centrifuge.subscribe("news", function(message) {
    // handle message
});

subscription.on("subscribe", function(sub) {
    sub.publish({"input": "hello world"}, function() {
        // success ack from Centrifugo received
    }, function(err) {
        // publish call failed with error
    });
});
```

### join and leave events

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

### unsubscribe

You can call `unsubscribe` method to unsubscribe from subscription:

```javascript
subscription.unsubscribe();
```

### disconnect event

`disconnect` event fired on centrifuge object every time client disconnects for
some reason. This can be network disconnect or disconnect initiated by Centrifugo server.

```javascript
centrifuge.on('disconnect', function() {
    // do whatever you need in case of disconnect from server
});
```

### error event

`error` event called every time on centrifuge object when error occurred, in normal
cases it will never be called. If it was called then your client does something wrong
or forbidden or maybe Centrifugo server error happened.

```javascript
centrifuge.on('error', function(error) {
    // handle error
});
```

### disconnect

In some cases you may need to disconnect your client from server, use `disconnect` method to
do this:

```javascript
centrifuge.disconnect();
```

After calling this client will not try to reestablish connection periodically. You must call
`connect` method manually again.

### message batching

There is also message batching support. It allows to send several messages to server in one request - this
can be especially useful when connection established via one of SockJS polling transports.

You can start collecting messages to send calling `startBatching()` method:

```javascript
centrifuge.startBatching();
```

When you want to actually send all collected messages to server call `flush()` method:

```javascript
centrifuge.flush();
```

Maximum amount of messages in one batching request is 100 (this is by default and can be changed
in server configuration).

Finally if you don't want batching anymore call `stopBatching()` method:


```javascript
centrifuge.stopBatching();
```

Call `stopBatching(true)` to flush all messages and stop batching:

```javascript
centrifuge.stopBatching(true);
```

### private channels

If channel name starts with `$` then subscription on this channel will be checked via AJAX POST request from
javascript to your web application.

You can subscribe on private channel as usual:

```javascript
centrifuge.subscribe('$private', function(message) {
    // process message
});
```

But in this case client will first check subscription via your backend sending POST request
to `/centrifuge/auth` endpoint (by default). This request will contain `client` parameter
which is your connection client ID and `channels` parameter - one or multiple private channels
client wants to subscribe to. Your server should validate all this subscriptions and return
properly signed responses.

There are also two new public API methods which can help to subscribe to many private
channels sending only one POST request to your web application backend: `startAuthBatching()`
and `stopAuthBatching()`. When you `startAuthBatching()` javascript client will collect
private subscriptions until `stopAuthBatching()` call - and then send them all at once.

Read more about private channels in [special documentation chapter](../mixed/private_channels.md).
