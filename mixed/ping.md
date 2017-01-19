# Ping messages

This document describes how Centrifugo works with client-server pings.

In internet environment ping messages required for keeping long-living connections
alive. Centrifugo is a server that works with such connections - so ping messages is
what we pay a lot of attention to.

Starting from Centrifugo v1.6.0 `centrifuge-js` sends periodic ping messages to Centrifugo
automatically. It works in a manner that ping messages only sent when connection is idle - i.e.
there was no activity for some time (by default 25 seconds). This means that ping messages do
not introduce significant overhead for busy applications.

Javascript client allows you to disable automatic ping messages - see its documentation.

Client to server pings allow to detect problems with connection and disconnect in case of no
response from server received in reasonable time.

Centrifugo also sends server to client pings every 25 seconds. In case of raw websocket it's
a special PING websocket frame, in case of SockJS it's an `h` frame. Server to client pings
allow to close inactive connections.

In short - if you use default settings - you are ok and don't need to worry.
