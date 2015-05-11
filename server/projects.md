# Projects

Project structure contains an array of projects. Project is an object with 2 required fields: `name` and `secret`.

So the simplest project structure looks like this:

```javascript
{
    "projects": [
        {
            "name": "bananas",
            "secret": "very-long-secret-key-for-bananas-project"
        }
    ]
}
```

* `name` – unique name of project (name must satisfy regexp `^[-a-zA-Z0-9_]{2,}$`). Used as project key when sending API requests and must be provided by clients when they want to connect to Centrifugo from browser.
* `secret` – secret key for project. The only two who should know this secret key is Centrifugo itself and your web application backend. It used to generate client connection tokens (more about them later), sign API requests and sign private channel subscription requests.

The next option is `connection_lifetime`:

* `connection_lifetime` – time in seconds for client connection to expire. By default it equals `0` - this means that connection can live forever and will not expire. See more about connection expiration mechanism in special chapter.

All other project options related to channels. Channel is an entity to which clients can subscribe to receive messages
published into that channel. Channel is just a string - but several symbols has special meaning - see next section to get
more information about channels. But now you should only understand that th following project options will affect channel
settings for this project:

* `watch` – Centrifugo will additionally publish messages into admin channel (these messages can be visible in
    web interface). Turn it off if you are expecting high message rate in channels. By default `false`.

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
    minimum. By default history size is `0` - this means that channels will have no history messages at all.

* `history_lifetime` - interval in seconds how long to keep channel history messages. As all
    history is storing in memory it is also very important to get rid of old history data
    for unused (inactive for a long time) channels. By default history lifetime is `0` – this
    means that channels will have no history messages at all. So to get history messages you
    should wisely configure both `history_size` and `history_lifetime` options.


