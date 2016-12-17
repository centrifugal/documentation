# Websocket compression

This is an experimental feature for raw websocket endpoint - `permessage-deflate` compression for
websocket messages. Btw look at [great article](https://www.igvita.com/2013/11/27/configuring-and-optimizing-websocket-compression/) about websocket compression.

We consider this experimental because this websocket compression is experimental in [Gorilla Websocket](https://github.com/gorilla/websocket) library that Centrifugo uses internally.

Websocket compression can reduce amount of traffic travelling over wire. But keep in mind that
enabling websocket compression can result in slower Centrifugo performance - depending on your
load this can be noticeable.

To enable websocket compression for raw websocket endpoint set `websocket_compression: true` in
configuration file. After this clients that support permessage-deflate will negotiate compression
with server automatically. Note that enabling compression does not mean that every connection will
use it - this depends on client support for this feature.

Another option is `websocket_compression_min_size`. Default 0. This is a minimal size of message
in bytes for which we use `deflate` compression when writing it to client's connection. Default
value `0` means that we will compress all messages when `websocket_compression` enabled and
compression support negotiated with client.
