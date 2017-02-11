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

## Advanced endpoint configuration.

After you started Centrifugo you have several endpoints available. As soon as you have not provided any extra options you have 3 endpoints by default.

#### Default endpoints.

First is SockJS endpoint - it's needed to serve client connections that use SockJS library:

```
http://localhost:8000/connection
```

Next is raw Websocket endpoint to serve client connections that use pure Websocket protocol:

```
ws://localhost:8000/connection/websocket
```

And finally you have API endpoint to `publish` messages to channels (and execute other available API commands):

```
http://localhost:8000/api/
```

By default all endpoints work on port `8000`. You can change it using `port` option:

```
{
    "port": "9000"
}
```

In production setup you will have your domain name in endpoint addresses above instead of `localhost`. Also if your Centrifugo will be behind proxy or load balancer software you most probably won't have ports in your
endpoint addresses. What will always be the same as shown above are URL paths: `/connection`, `/connection/websocket`, `/api/`.

Let's look at possibilities to tweak available endpoints.

#### Admin endpoints.

First is enabling admin endpoints:

```
{
    ...
    "admin": true,
    "admin_password": "password",
    "admin_secret": "secret"
}
```

This makes the following endpoint available:

```
ws://localhost:8000/socket
```

This is an endpoint for admin websocket connections. In most scenarios it's used only by our builtin web
interface. You can read about web interface in dedicated chapter. Here we will just show how to enable it:

```
{
    ...
    "web": true,
    "admin": true,
    "admin_password": "password",
    "admin_secret": "secret"
}
```

After adding `web` option you can visit:

```
http://localhost:8000/
```

And see web interface. You can log into it using `admin_password` value we set above.


#### Debug endpoints.

Next, when Centrifugo started in debug mode some extra debug endpoints become available.
To start in debug mode add `debug` option to config:

```
{
    ...
    "debug": true
}
```

And endpoint:

```
http://localhost:8000/debug/pprof/
```

will show you useful info about internal state of Centrifugo instance. This info is especially helpful when troubleshooting.

#### Custom admin and API ports

We strongly recommend to not expose admin (web), debug and API endpoints to internet. In case of admin endpoints this step
provides extra protection to `/socket` endpoint, web interface and debug endpoints. Protecting API endpoint will allow you to use `insecure_api`
mode to omit signing of each API request.

So it's a good practice to protect admin and API endpoints with firewall. For example you can do this in `location` section of Nginx configuration.

Though sometimes you don't have access to per-location configuration in your proxy/load balancer software. For example
when using Amazon ELB. In this case you can change ports on which your admin and API endpoints work.

To run admin endpoints on custom port use `admin_port` option:

```
{
    ...
    "admin_port": "10000"
}
```

So admin socket will work on address:
 
```
ws://localhost:10000/socket
```

And debug page will be available on new custom admin port too:

```
http://localhost:10000/debug/pprof/
```

To run API server on it's own port use `api_port` option:

```
{
    ...
    "api_port": "10001"
}
```

Now you should send API requests to:

```
http://localhost:10001/api/
```
