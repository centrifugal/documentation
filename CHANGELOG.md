# Remove multiple project support

At this stage multiple project support was removed. Centrifugo server now works with only one application.

If you are looking for documentation of version with multiple projects support - look at [docs v0.3.0](https://github.com/centrifugal/documentation/tree/v0.3.0).

Here is a list of libraries versions compatible with Centrifugo with multiple projects support:

* centrifuge-js [0.9.0](https://github.com/centrifugal/centrifuge-js/tree/0.9.0)
* cent [v0.6.0](https://github.com/centrifugal/cent/tree/v0.6.0)
* adjacent [v0.3.0](https://github.com/centrifugal/adjacent/tree/v0.3.0)
* web [v0.1.0](https://github.com/centrifugal/web/tree/v0.1.0)
* examples [v0.1.0](https://github.com/centrifugal/examples/tree/v0.1.0)
* phpcent [0.6.1](https://github.com/centrifugal/phpcent/tree/0.6.1)
* centrifuge-ruby [v0.1.0](https://github.com/centrifugal/centrifuge-ruby/tree/v0.1.0)


# Full changelog of Centrifugal

This is an aggregated Centrifugal changelog from all important repositories to help with updating.

Adjacent v0.3.0
===============

* support `channels` command (Centrifugo >= 0.3.0 required)

Cent v0.6.0
===========

* support `channels` command (Centrifugo >= 0.3.0 required)

Centrifugo v0.3.0
=================

* new `channels` API command – allows to get list of active channnels in project at moment (with one or more subscribers).
* `message_send_timeout` option default value is now 0 (last default value was 60 seconds) i.e. send timeout is not used by default. This means that Centrifugo won't start lots of goroutines and timers for every message sent to client. This helps to drastically reduce memory allocations. But in this case it's recommended to keep Centrifugo behind properly configured reverse proxy like Nginx to deal with connection edge cases - slow reads, slow writes etc.
* Centrifugo now sends pings into pure Websocket connections. Default value is 25 seconds and can be adjusted using `ping_interval` configuration option. Note that this option also sets SockJS heartbeat messages interval. This opens a road to set reasonable value for Nginx `proxy_read_timeout` for `/connection` location to mimic behaviour of `message_send_timeout` which is now not used by default
* improvements in Redis Engine locking.
* tests now require Redis instance running to test Redis engine. Tests use Redis database 9 to run commands and if that database is not empty then tests will fail to prevent corrupting existing data.
* all dependencies now vendored.

Centrifugo v0.2.4
=================

* HTTP API endpoint now can handle json requests. Used in client written in Go at moment. Old behaviour have not changed, so this is absolutely optional.
* dependency packages updated to latest versions - websocket, securecookie, sockjs-go.

Centrifuge-js v0.9.0
====================

The only change in `0.9.0` is changing private channel request POST parameter name
`channels` to `channels[]`. If you are using private channels then you should update
your backend code to fit new parameter name. This change was required because of how
PHP and Ruby on Rails handle POST parameter names when POST request contains multiple
values for the same parameter name.

* `channels` parameter renamed to `channels[]` in private subscription POST request to application.

Cent v0.5.0
==========

Added several API methods for client to simplify sending single commands to API.

* `client.publish(channel, data, client=None)`
* `client.presence(channel)`
* `client.history(channel)`
* `client.unsubscribe(user_id, channel=None)`
* `client.disconnect(user_id)`

For example:

```python
from cent.core import Client

client = Client("http://localhost:8000", "development", "secret")

for i in range(1000):
    res, err = client.publish("$public:docs", {"json": True})
    print res
```

Centrifugo v0.2.3
=================

Critical bug fix for Redis Engine!

* fixed bug when entire server could unsubscribe from Redis channel when client closed its connection.


Centrifugo v0.2.2
=================

* Add TLS support. New flags are:
  * `--ssl`                   - accept SSL connections. This requires an X509 certificate and a key file.
  * `--ssl_cert="file.cert"`  - path to X509 certificate file.
  * `--ssl_key="file.key"`    - path to X509 certificate key.
* Updated Dockerfile

Centrifugo v0.2.1
=================

* set expire on presence hash and set keys in Redis Engine. This prevents staling presence keys in Redis.

Centrifugo v0.2.0
=================

* add optional `client` field to publish API requests. `client` will be added on top level of
	published message. This means that there is now a way to include `client` connection ID to
	publish API request to Centrifugo (to get client connection ID call `centrifuge.getClientId()` in
	javascript). This client will be included in a message as I said above and you can compare
	current client ID in javascript with `client` ID in message and if both values equal then in
	some situations you will wish to drop this message as it was published by this client and
	probably already processed (via optimistic optimization or after successful AJAX call to web
	application backend initiated this message).
* client limited channels - like user limited channels but for client. Only client with ID used in
	channel name can subscribe on such channel. Default client channel boundary is `&`. If you have
	channels with `&` in its name - then you must adapt your channel names to not use `&` or run Centrifugo with another client channel boundary using `client_channel_boundary` configuration
	file option.
* fix for presence and history client calls - check subscription on channel before returning result
	or return Permission Denied error if not subscribed.
* handle interrupts - unsubscribe clients from all channels. Many thanks again to Mr Klaus Post.
* code refactoring, detach libcentrifugo real-time core from Centrifugo service.

Centrifugo v0.1.1
=================

Lots of internal refactoring, no API changes. Thanks to Mr Klaus Post (@klauspost) and Mr Dmitry Chestnykh (@dchest)

Centrifugo v0.1.0
=================

First release. New [documentation](http://fzambia.gitbooks.io/centrifugal/content/).