# Insecure mode

You can run Centrifugo in insecure mode.

Insecure mode:

* disables client timestamp and token check
* allows anonymous access for all channels
* allows client to publish into all channels
* suppresses connection check

This allows to use Centrifugo and centrifuge javascript client as a quick and simple
solution when making real-time demos, presentations, testing ideas etc. But this mode
is mostly for personal and demonstration uses - you should never turn this mode on
in production until you really want it to be there.

### on server side

To start Centrifugo in this mode use `--insecure` flag:

```bash
centrifuge --config=config.json --insecure
```

You can also set `insecure` option in configuration file to do the same.

### on client side

When using insecure mode you can create client connection in this way:

```javascript
var centrifuge = new Centrifuge({
    "url": url,
    "project": project,
    "insecure": true
});
```

I.e. without `token`, `user` and `timestamp` parameters. So you can connect to
Centrifugo without any backend code.

Look at [demo](https://github.com/centrifugal/centrifuge/tree/master/examples/insecure_mode) to
see insecure mode in action.
