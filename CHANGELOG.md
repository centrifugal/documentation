# Full changelog of Centrifugal

This is an aggregated Centrifugal changelog from all important repositories to help with updating.

Centrifugo v1.3.0
=================

Possible backwards incompatibility here (in client side code) - see first point.

* omit fields in message JSON if field contains empty value: `client` on top level, `info` on top level, `default_info` in `info` object, `channel_info` in `info` object. This also affects top level data in join/leave messages and presence data – i.e. `default_info` and `channel_info` keys not included in JSON when empty. This can require adapting your client side code a bit if you rely on these keys but for most cases this should not affect your application. But we strongly suggest to test before updating. This change allows to reduce message size. See migration notes below for more details.
* new option `--admin_port` to bind admin websocket and web interface to separate port. [#44](https://github.com/centrifugal/centrifugo/issues/44)
* new option `--api_port` to bind API endpoint to separate port. [#44](https://github.com/centrifugal/centrifugo/issues/44)
* new option `--insecure_web` to use web interface without setting `web_password` and `web_secret` (for use in development or when you protected web interface by firewall rules). [#44](https://github.com/centrifugal/centrifugo/issues/44)
* new channel option `history_drop_inactive` to drastically reduce resource usage (engine memory, messages travelling around) when you use message history. See [#50](https://github.com/centrifugal/centrifugo/issues/50)
* new Redis engine option `--redis_api_num_shards`. This option sets a number of Redis shard queues Centrifugo will use in addition to standard `centrifugo.api` queue. This allows to increase amount of messages you can publish into Centrifugo and preserve message order in channels. See [#52](https://github.com/centrifugal/centrifugo/issues/52) and [documentation](https://fzambia.gitbooks.io/centrifugal/content/server/engines.html) for more details.
* fix race condition resulting in client disconnections on high channel subscribe/unsubscribe rate. [#54](https://github.com/centrifugal/centrifugo/issues/54)
* refactor `last_event_id` related stuff to prevent memory leaks on large amout of channels. [#48](https://github.com/centrifugal/centrifugo/issues/48)
* send special disconnect message to client when we don't want it to reconnect to Centrifugo (at moment to client sending malformed message).
* pong wait handler for raw websocket to detect non responding clients.

See [Changelog](https://github.com/centrifugal/centrifugo/blob/master/CHANGELOG.md#v130) in Centrifugo repo for more migration details.


Centrifuge-js 1.2.0
===================

* use exponential backoff when reconnecting
* follow disconnect advice from Centrifugo


Centrifugo v1.2.0
=================

No backwards incompatible changes here.

* New `recover` option to automatically recover missed messages based on last message ID. See [pull request](https://github.com/centrifugal/centrifugo/pull/42) and [chapter in docs](https://fzambia.gitbooks.io/centrifugal/content/server/recover.html) for more information. Note that you need centrifuge-js >= v1.1.0 to use new `recover` option
* New `broadcast` API method to send the same data into many channels. See [issue](https://github.com/centrifugal/centrifugo/issues/41) and updated [API description in docs](https://fzambia.gitbooks.io/centrifugal/content/server/api.html)
* Dockerfile now checks SHA256 sum when downloading release zip archive.
* release built using Go 1.5.2

Centrifugo v1.1.0
=================

No backwards incompatible changes here.

* support enabling web interface over environment variable CENTRIFUGO_WEB
* close client's connection after its message queue exceeds 10MB (default, can be modified using `max_client_queue_size` configuration file option)
* fix theoretical server crash on start when reading from redis API queue

Cent v1.1.0
===========

Some improvements in `cent` public API here.

`publish`, `unsubscribe`, `disconnect` helper methods now return just an error if any error occurred.

`presence`, `history`, `stats`, `channels` helper methods now return data requested and error
(instead of full response and error). `error` field of response now wrapped in `ResponseError`
exception. This means that now you don't need to extract response body and then data from it and
check response error manually in your code every time you use methods above.

For example see calling `stats` method:

```python
from cent.core import Client

client = Client("http://localhost:8000", "secret")

stats, error = client.stats()
if error:
    # error occurred, handle it in a way you prefer.
    raise error

print stats
```

Compare with code required before to do the same:

```python
from cent.core import Client

client = Client("http://localhost:8000", "secret")

resp, error = client.stats()
if error:
    # error occurred, handle it
    raise error
if resp["error"]:
    # handle response error
    raise Exception(resp["error"])

stats = resp["body"]["data"]
print stats
```

I.e. here is how to use helper methods:

```python
from cent.core import Client

client = Client("http://localhost:8000", "secret")

error = client.publish("public:chat", {"input": "test"})
error = client.unsubscribe("user_id_here")
error = client.disconnect("user_id_here")
messages, error = client.history("public:chat")
clients, error = client.presence("public:chat")
channels, error = client.channels()
stats, error = client.stats()
```

Low level sending over calling `add` method not affected in this release.


Adjacent v1.0.0
===============

* support for Centrifugo 1.0.0

This means that no more project key required.

Just set:

```
CENTRIFUGE_ADDRESS = 'http://centrifuge.example.com'
CENTRIFUGE_SECRET = 'your secret key from Centrifugo'
CENTRIFUGE_TIMEOUT = 10
```

And you are done.


Cent v1.0.0
===========

* support for Centrifugo 1.0.0

This means that no more project key required.

How to migrate
--------------

Omit project key when instantiating Client:

```
from cent.core import Client
client = Client("http://localhost:8000", "project_secret")
```

And also note that token and sign generation function do not accept project key anymore.


Centrifuge-js 1.0.0
===================

One backwards incompatible change here. Centrifuge-js now sends JSON (`application/json`)
request instead of `application/x-www-form-urlencoded` when client wants to subscribe on
private channel. See [in docs](https://fzambia.gitbooks.io/centrifugal/content/mixed/private_channels.html) how to deal with JSON in this case.

* send JSON instead of form when subscribing on private channel.
* simple reconnect strategy

Centrifugo v1.0.0
=================

A bad and a good news here. Let's start with a good one. Centrifugo is still real-time messaging server and just got v1.0 release. The bad – it is not fully backwards compatible with previous versions. Actually there are three changes that ruin compatibility. If you don't use web interface and private channels then there is only one change. But it affects all stack - Centrifugo itself, client library and API library.

Starting from this release Centrifugo won't support multiple registered projects. It will work with only one application. You don't need to use `project key` anymore. Changes resulted in simplified
configuration file format. The only required option is `secret` now. See updated documentation
to see how to set `secret`. Also changes opened a way for exporting Centrifugo node statistics via HTTP API `stats` command.

As this is v1 release we'll try to be more careful about backwards compatibility in future. But as we are trying to make a good software required changes will be done if needed.

Highlights of this release are:

* Centrifugo now works with single project only. No more `project key`. `secret` the only required configuration option.
* web interface is now embedded, this means that when downloading release you get possibility to run Centrifugo web interface just providing `--web` flag to `centrifugo` when starting process.
* when `secret` set via environment variable `CENTRIFUGO_SECRET` then configuration file is not required anymore. But note, that when Centrifugo configured via environment variables it's not possible to reload configuration sending HUP signal to process.
* new `stats` command added to export various node stats and metrics via HTTP API call. Look its response example [in docs chapter](https://fzambia.gitbooks.io/centrifugal/content/server/api.html).
* new `insecure_api` option to turn on insecure HTTP API mode. Read more [in docs chapter](https://fzambia.gitbooks.io/centrifugal/content/mixed/insecure_mode.html).
* minor clean-ups in client protocol. But as protocol incapsulated in javascript client library you only need to update centrifuge-js.
* release built using Go 1.5.1

[Documentation](https://fzambia.gitbooks.io/centrifugal/content/) was updated to fit all these release notes. Also all API and client libraries were updated – Javascript browser client (`centrifuge-js`), Python API client (`cent`), Django helper module (`adjacent`). API clients for Ruby (`centrifuge-ruby`) and PHP (`phpcent`) too. Admin web interface was also updated to support changes introduced here.

There are 2 new API libraries: [gocent](https://github.com/centrifugal/gocent) and [jscent](https://github.com/centrifugal/jscent). First for Go language. And second for NodeJS.

Also if you are interested take a look at [centrifuge-go](https://github.com/centrifugal/centrifuge-go) – experimental Centrifugo client for Go language. It allows to connect to Centrifugo from non-browser environment. Also it can be used as a reference to make a client in another language (still hoping that clients in Java/Objective-C/Swift to use from Android/IOS applications appear one day – but we need community help here).

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