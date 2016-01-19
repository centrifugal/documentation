# Channels

Channel is a route for messages.

Clients can subscribe on channel to receive events related to this channel - new
messages, join/leave events etc. Also client must be subscribed on channel to get
presence or history information.

Channel is just a string - ``news``, ``comments`` are valid channel names.

**BUT!** You should remember several things.

First, channel name length is limited by `255` characters by default (can
be changed via configuration file option `max_channel_length`)

Second, `:`, `#`, `&` and `$` symbols have a special role in channel name.

### namespace channel boundary

``:`` - is a channel namespace boundary.

If channel is `public:chat` - then Centrifuge will apply options to this channel
from channel namespace with name `public`.

### user channel boundary

`#` is a user boundary - separator to create private channels for users (user limited
channels)without sending POST request to your web application. For example if channel
is `news#42` then only user with ID `42` can subscribe on this channel (Centrifugo
knows user ID as clients provide it when connecting).

Moreover you can provide several user IDs in channel name separated by comma: `dialog#42,43` –
in this case only user with ID `42` and user with ID `43` will be able to subscribe on this channel.

### client channel boundary (new in 0.2.0)

`&` is a client channel boundary. This is similar to user channel boundary but limits
access to channel only for one client with ID set after `&`. For example if channel is
`client&7a37e561-c720-4608-52a8-a964a9db7a8a` then only client with client ID
`7a37e561-c720-4608-52a8-a964a9db7a8a` (call `centrifuge.getClientId()` in javascript to
get client's ID) will be able to subscribe on this channel.

### private channel prefix

If channel starts with `$` then it considered private. Subscription on private channel
must be properly signed by your web application. Read special chapter in docs about
private channel subscriptions.
