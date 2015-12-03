# How recover mechanism works

`recover` option available since v1.2.0

This option inspired by Last-Event-ID mechanism [from Eventsource protocol](http://www.w3.org/TR/2012/WD-eventsource-20120426/).

Let's describe motivation. We live in non-ideal world and network connection can
disappear sometimes. While client offline some messages could be sent into channel
and when client goes online those messages are lost. Centrifugo can help to automatically
recover reasonable amount of missed messages.

There are still cases when Centrifugo can't help – for example if client was disconnected
for a week or tons of messages were sent into channels since last subscription attempt...
In these situations client must recover its state with application frontend/backend code help.

But when connection disappeared for a short amount of time then Centrifugo can automatically
recover missed messages when client resubscribes on channel. For this purpose client provides
last message ID seen when resubscribing and Centrifugo automatically tries to recover missed
messages from message history.

Recover option must be used in combination with `history_size` and `history_lifetime`. Both
`history_size` and `history_lifetime` must be reasonably configured and recover turned on for
channel (for all channels or for namespace - you decide).

Note that sometimes your clients don't need to get missed messages at all. This depends on
nature of real-time messages you publish. For example you really don't need missed messages
in case of frequently updated counter with actual value in message. So developer must think
wisely when he wants to enable this mechanism. For example it can be reasonable to enable
it if you show comments on page in real-time. In this case when your client goes online it
receives all missed comments automatically from Centrifugo message history.

Also note that there are other possible solutions to get missed messages based on your
application code - you can still manually retrieve message history from Centrifugo or from
your application backend.

Here I'll describe how `recover` option implemented based on interaction between our javascript
client and Centrifugo server. You don't need to read this to just use `recover` feature
as all logic below encapsulated into centrifuge-js client.

After client first subscribes on channels it doesn't need any missed messages because he
didn't miss anything yet. So it sends subscription command to Centrifugo which contains
channel and `recover` flag set to `false` to indicate that Centrifugo must not search for
missed messages in message history.

```
{
    "channel": "news",
    "recover": false
}
```

Centrifugo subscribes client on channel `news` and answers back to client with subscribe response.
That response includes field `last` in response body containing last message ID for channel `news`
that Centrifugo has.

```
{
    "last": "last-message-id-for-channel"
}
```

Client saves that last message ID for channel and start listening for messages in channel `news`.
When new message arrives client saves message `uid` value and that value considered as last message
ID for channel `news`. So when client lost network connection and the resubscribes on channel `news`
he can provide last message ID in subscription request.

```
{
    "channel": "news",
    "recover": true,
    "last": "last-message-id-for-channel"
}
```

Centrifugo receives this subscription requests and tries to find all missed messages looking
into message history. And then returns all missed messages to client in subscription response body:

```
{
    "messages": [message, message...]
}
```

So client can process those missed messages and continue to listen channel `news` for new
published messages.
