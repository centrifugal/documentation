# Full changelog of Centrifugal

This is an aggregated Centrifugal changelog from all important repositories to help with updating.

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