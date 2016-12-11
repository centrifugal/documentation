# Centrifugo best practice advices.

This is a collection of advices that could be useful if you want to use Centrifugo.

### Publish new messages from your application backend

Centrifugo designed to stream messages from server to client. Even though it's possible to
publish messages into channels directly from client (when `publish` channel option enabled) -
we strongly discourage this in production usage as those messages will go through Centrifugo
without any control.

Sometimes this can be useful though - for personal projects, for demonstrations (like we do
in our [examples](https://github.com/centrifugal/examples)) or if you trust your users and want
to build application without backend.

But in general when user generates an event it must be first delivered to your app backend using
a convenient way (for example AJAX POST request for web application), processed on backend (validated,
saved into main database) and then published to Centrifugo using Centrifugo HTTP API or Redis queue.

### Use namespaces for channel configuration

In most situations your application need several real-time features. We suggest to use
namespaces for every real-time feature if it requires some option enabled.

For example if you need join/leave messages for chat app - create special channel namespace
with this `join_leave` option enabled. Otherwise your other channels will receive join/leave messages
that just increase load and traffic in system but not used by clients.

The same relates to other channel options.

### Centrifugo is a best-effort transport

This means that if you want strongly guaranteed message delivery to your clients then
you can't just rely on Centrifugo and its message history cache. Centrifugo delivers
messages at most once.

In this case you still can use Centrifugo for real-time but you should build some additional
logic on top your application backend and main data storage to satisfy your guarantees.