# Websocket compression

This is an experimental feature for raw websocket endpoint - `permessage-deflate` compression for
websocket messages.

We consider this experimental because this websocket compression is experimental in Gorilla Websocket
library that Centrifugo uses internally.

Websocket compression can reduce amount of traffic travelling over wire. But it can cost some
performance penalty on server as messages will be compressed.

To enable websocket compression for raw websocket endpoint set `websocket_compression: true` in
configuration file. After this clients that support permessage-deflate will negotiate compression
with server automatically. Note that enabling compression does not mean that every connection will
use it - this depends on client support for this feature.
