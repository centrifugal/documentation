# Use of SockJS 0.3.4 instead of 1.0

At moment SockJS 1.0 is used on client and server sides.

If you want to use SockJS 0.3.4 then you need to make 2 steps.

### on server side

Set `sockjs_url` in configuration file to point SockJS client library you are planning
to use on client side:

`config.json`:

```
{
    ...,
    "sockjs_url": "https://cdn.jsdelivr.net/sockjs/0.3.4/sockjs.min.js"
}
```

### on client side

Use SockJS 0.3.4 instead of 1.0 version. But use `protocols_whitelist` instead of
`transports` when setting Centrifuge instance options.

Support for SockJS 0.3.4 may be dropped in future releases so I highly recommend to
use version 1.0