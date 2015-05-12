# Use of SockJS 1.0.0

At moment SockJS 0.3.4 used on client and server sides.

SockJS 1.0.0 is currently in beta but it's already possible to use it.

### on server side

Set `sockjs_url` in configuration file to point SockJS client library you are planning
to use on client side:

`config.json`:

```
{
    ...,
    "sockjs_url": "https://rawgit.com/sockjs/sockjs-client/master/dist/sockjs-1.0.0-beta.13.min.js"
}
```

### on client side

Use SockJS 1.0.0 instead of 0.3.4. But use `transports` instead of `protocols_whitelist` when
setting Centrifuge instance options.