# Advanced configuration

Centrifugo has some options for which default values make sense for most applications. In many case you
don't need (and you really should not) change them. This chapter is about such options.

#### client_channel_limit

Default: 128

Sets maximum number of different channel subscriptions single client can have.

Before Centrifugo v1.6.0 default value was 100.

#### max_channel_length

Default: 255

Sets maximum length of channel name.

#### user_connection_limit

Default: 0

Maximum number of connections from user (with known ID) to Centrifugo node. By default - unlimited.

#### node_metrics_interval

Default: 60

Interval in seconds Centrifugo aggregates metrics before making metrics snapshot.

#### client_request_max_size

Default: 65536

Maximum allowed size of request from client in bytes.

#### client_queue_max_size

Default: 10485760

Maximum client message queue size in bytes to close slow reader connections. By default - 10mb.

#### sockjs_heartbeat_delay

Default: 0

Interval in seconds how often to send SockJS h-frames to client. Starting from v1.6.0 we don't use hearbeat SockJS
frames as we use client to server pings.

#### websocket_compression

Default: false

Enable websocket compression, see special chapter in docs.

#### gomaxprocs

Default: 0

By default Centrifugo runs on all available CPU cores. If you want to limit amount of cores Centrifugo can utilize in one moment use this option.

There are some other internal undocumented options. But we don't think someone really need to tweak them.
