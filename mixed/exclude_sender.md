# Exclude message processing by sender.

In some situations you may want to prevent the client that published a message from processing
it after receiving from channel. The solution for doing this is described here.

### new in 0.2.0

Publish command includes optional `client` field. Include `client` ID (to get client connection
ID call `centrifuge.getClientId()` in javascript client) in publish API request and `client`
will be added on top level of published message. You can compare current client ID in javascript
with client ID in received message and if both values equal then in some situations you will wish
to drop this message as it was published by this client and probably already processed (via
optimistic optimization or after successful AJAX call to web application backend initiated this
message).

I.e. something like this:

```javascript
var subscription = centrifuge.subscribe(channel, function(message) {
    if (message.client === centrifuge.getClientId()) {
        return;
    }
    // if clients not equal – process message as usual
});
```

To include `client` into publish API request you should provide it for your backend
in AJAX data when client creates an event in your application.
