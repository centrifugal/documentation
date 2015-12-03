# Important configuration settings

As I wrote in previous chapter configuration file must have one required option: `secret`.

The configuration file looks like this:

```javascript
{
    "secret": "very-long-secret-key"
}
```

The only two who should know this secret key is Centrifugo itself and your web application
backend. It used to generate client connection tokens (more about them later), sign API
requests and sign private channel subscription requests.

The next available option is `connection_lifetime`:

`connection_lifetime` – time in seconds for client connection to expire. By default it
equals `0` - this means that connection can live forever and will not expire.
See more about connection expiration mechanism in special chapter. In most situations
you don't have to explicitly set it as no connection expiration is ok for most of
web applications.

```javascript
{
    "secret": "very-long-secret-key",
    "connection_lifetime": 0
}
```

Let's look on other options related to channels. Channel is an entity to which clients can subscribe
to receive messages published into that channel. Channel is just a string - but several symbols has
special meaning - see section about channels to get more information about channels. But now you should
only understand that the following options will affect channel behaviour:

* `watch` – Centrifugo will additionally publish messages into admin channel (these
    messages can be visible in web interface). Turn it off if you are expecting high
    message rate in channels. By default `false`.

* `publish` – allow clients to publish messages into channels directly. Your web application never
    receive those messages. In general all messages must be published by your web application backend.
    But this option can be useful when you want something without backend-side validation and saving
    into your database. By default `false`.

* `anonymous` – allow anonymous (with empty USER) clients to subscribe on channels. By default `false`.

* `presence` – enable/disable presence information. By default `false`.

* `join_leave` – enable/disable sending join(leave) messages when client subscribes on
    channel (unsubscribes from channel). By default `false`.

* `history_size` – history size (amount of messages) for channels. As all messages stored in process
    memory it's very important to limit maximum amount of messages in channel history to reasonable
    minimum. By default history size is `0` - this means that channels will have no history messages
    at all.

* `history_lifetime` – interval in seconds how long to keep channel history messages. As all
    history is storing in memory it is also very important to get rid of old history data
    for unused (inactive for a long time) channels. By default history lifetime is `0` – this
    means that channels will have no history messages at all. So to get history messages you
    should wisely configure both `history_size` and `history_lifetime` options.

* `recover` (**new in v1.2.0**) – boolean option, when enabled Centrifugo will try to recover
    missed messages published while client was disconnected for some reason (bad internet
    connection for example). By default `false`. This option must be used in conjunction with
    reasonably configured message history for channel i.e. `history_size` and `history_lifetime`
    must be set (because Centrifugo uses channel message history to recover messages). Also
    note that note all real-time events require this feature turned on so think wisely when
    you need this. See more details about how this option works in [special chapter](recover.md).

Let's look how to set all of these options in config:

```javascript
{
    "secret": "very-long-secret-key",
    "connection_lifetime": 0,
    "anonymous": true,
    "publish": true,
    "watch": true,
    "presence": true,
    "join_leave": true,
    "history_size": 10,
    "history_lifetime": 30
}
```

And the last channel specific option is `namespaces`. `namespaces` are optional and if set must
be an array of namespace objects. Namespace allows to configure custom options for channels starting with
namespace name. This provides a great control over channel behaviour.

Namespace has a name and the same channel options (with same defaults) as described above.

* `name` - unique namespace name (name must must consist of letters, numbers, underscores
    or hyphens and be more than 2 symbols length i.e. satisfy regexp `^[-a-zA-Z0-9_]{2,}$`).

If you want to use namespace options for channel - you must include namespace name into
channel name with `:` as separator:

`public:messages`

`gossips:messages`

Where `public` and `gossips` are namespace names from project `namespaces`.

All things together here is an example of `config.json` which includes registered
project with all options set and 2 additional namespaces in it:

```javascript
{
    "secret": "very-long-secret-key",
    "connection_lifetime": 0,
    "anonymous": true,
    "publish": true,
    "watch": true,
    "presence": true,
    "join_leave": true,
    "history_size": 10,
    "history_lifetime": 30,
    "namespaces": [
        {
          "name": "public",
          "publish": true,
          "presence": true,
          "join_leave": true,
          "anonymous": true,
          "history_size": 10,
          "history_lifetime": 30
        },
        {
          "name": "gossips",
          "watch": true
        }
    ]
}
```

Channel `news` will use global project options.

Channel `public:news` will use `public` namespace's options.

Channel `gossips:news` will use `gossips` namespace's options.
