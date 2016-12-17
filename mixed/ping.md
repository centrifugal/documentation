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

In version 1.6.0 Centrifugo relies on client to server pings. SockJS `h` heartbeat frames now
disabled by default (because for polling transports it's an additional reconnect just after client
sent its own ping to server). It's possible to turn it on though: use `sockjs_heartbeat_delay: 25`
to restore old behaviour. If you disabled automatic `ping` in Javascript client and want to use
SockJS you must set `sockjs_heartbeat_delay` or Centrifugo won't be able to close inactive
connections. For raw websocket Centrifugo still sends server to client ping websocket frames so
even without client to server ping messages it's able to close idle connections.

In short - if you use default settings - you are ok and don't need to worry.

Client implementations can help Centrifugo to understand that client will send pings to server
periodically. Adding `"ping": true` to `connect` command will do the work. Centrifugo will close
such connections after configurable period of inactivity (`client_max_idle_timeout` option, default
60 seconds).
