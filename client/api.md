# Client API methods

When centrifuge object properly initialized then it is ready to start communicating with server.

First, we should actually `connect` to server endpoint:

```javascript
var centrifuge = new Centrifuge({
    url: 'CONNECTION ENDPOINT',
    project: 'PROJECT KEY',
    user: 'USER ID',
    timestamp: 'UNIX TIMESTAMP',
    token: 'TOKEN'
});

centrifuge.connect();
```

`connect()` calls actual connection request to server with connection parameters you provided
during initialization.

After successful connect you can subscribe on channels. But you can only start subscribing when
connection with server was successfully established. If you try to subscribe on channel before
connection established - your subscription request will be rejected by server. There is an event
about successful connection and you can bind your subscription logic to it in this way:

```javascript
centrifuge.on('connect', function() {
    // now your client connected and authorized by server
});
```

Also you `disconnect` and `error` events available:

```javascript
    centrifuge.on('disconnect', function() {
        // do whatever you need in case of disconnect from server
    });

    centrifuge.on('error', function(errorMsg) {
        // called every time error occurred, in normal cases it will never be called, if it
        // was called then your client does something wrong or forbidden
    });
```

When your client connected, it is time to subscribe on channel:

```javascript
centrifuge.on('connect', function() {
    var subscription = centrifuge.subscribe('namespace:channel', function(message) {
        // called when new published message received from this channel
    });
}
```

If channel options allow publishing (`publish` option on) client can publish messages into such
channel. But as with connection you can not do it immediately after subscription request. Client
can only publish when `subscribe:success` (or its alias `ready`) event fired. The same for presence
and history requests. Lets publish message, get presence and get history data, subscribe on join/leave
events as soon as our subscription request returned successful subscription response:

```javascript
centrifuge.on('connect', function() {

    var subscription = centrifuge.subscribe('namespace:channel', function(message) {
        // called when new published message received from this channel
    });

    subscription.on('ready', function() {

        // publish into channel
        subscription.publish({"input": "hello"});

        // get presence information (who is currently subscribed on this channel)
        subscription.presence(function(message) {
            console.log("channel presence received");
            console.log(message);
        });

        // get history (last messages sent) for this channel
        subscription.history(function(message) {
            console.log("channel history received");
            console.log(message);
        });

        subscription.on('join', function(message) {
            // called when someone subscribed on channel
            console.log("someone subscribed on channel");
            console.log(message);
        });

        subscription.on('leave', function(message) {
            // called when someone unsubscribed from channel
            console.log("someone unsubscribed from channel");
            console.log(message);
        });

    });

}

You can `unsubscribe()` from subscription:

```javascript
subscription.unsubscribe();
```

In some cases you may need to `disconnect()` your client from server:

```javascript
centrifuge.disconnect();
```

After calling this client will not try to reestablish connection periodically. You must call
`connect` method manually.

There is also message batching support. It allows to send several messages to server in one request - this
can be especially useful when connection established via one of SockJS non-streaming transports.

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

Read more about private channels in special documentation chapter.
