# Server API

HTTP API is a way to send commands to Centrifugo. There is also another way to send commands –
using Redis engine but in this chapter we will talk mostly about HTTP API.

Why we need API?

If you look at project and namespace options you see option called `publish`. When turned on
this option allows browser clients to publish into channels directly. If client publishes a
message into channel directly - your application will not receive that message (it just goes
through Centrifugo towards subscribed clients). This pattern can be useful sometimes but in
most cases you first need to receive new event from client via AJAX, process it - probably
validate, save into database and then `publish` into Centrifugo using HTTP API and Centrifugo
will then broadcast message to all subscribed clients.

Also HTTP API can be used to send other types of commands - see all available commands below.

If your backend written in Python you can use [Cent](../libraries/python.md) API client. Also we have
client [for Ruby](../libraries/ruby.md). If you use other language don't worry - I will describe
how to communicate with Centrifugo HTTP API endpoint in this chapter.

Note also that there are 2 API endpoints in Centrifugo - first of them is HTTP API endpoint and
second – Redis Engine API. In this section we will talk about HTTP API mostly. You can find
description of Redis API in `engines` chapter - it has the same commands but simplified format.
So if you decide that HTTP API is too difficult and uncomfortable to use - you can use Redis API
to publish new messages into channels.

Let's start!

Centrifugo API url is `/api/<PROJECT_ID>`, where <PROJECT_ID> must be replaced with your project key
(=project name).

So if your Centrifugo sits on domain `https://centrifuge.example.com` and project key is `bananas`
then an API address will be `https://centrifuge.example.com/api/bananas`.

All you need to do to use HTTP API is to send correctly constructed POST request to this endpoint.

API request must have two POST parameters: `data` and `sign`.

`data` is a JSON string representing command (or commands) you want to send to Centrifugo
(see below) and `sign` is an HMAC based on project secret key, project key and JSON string
from `data`. This `sign` is used later by Centrifugo to validate API request.

`data` is a JSON string created from object with two properties: `method` and `params`.

`method` is a name of action you want to do.
`params` is an object with method arguments.

For example in Python

```
data = json.dumps({
    "method": "publish",
    "params": {"channel": "news", data:{}}
})
```

First lets see how to construct such request in Python. If Python is your language then you
don't have to implement this yourself as `Cent` python module exists. But this example here can
be useful for someone who want to implement interaction with Centrifugo API in language for
which we don't have API client yet.

Let's imagine that you have a project with name `development` and secret key `secret`. HTTP
API url then will be like `https://centrifuge.example.com/api/development`

```python
from urllib2 import urlopen, Request
from cent.core import generate_api_sign
import json

req = Request("http://localhost:8000/api/development")

commands = [
    {
        "method": "publish",
        "params": {"channel": "docs", "data": {"json": True}}
    }
]
encoded_data = json.dumps(commands)
sign = generate_api_sign("secret", "development", encoded_data)
data = urlencode({'sign': sign, 'data': encoded_data})
response = urlopen(req, data, timeout=5)
```

Here to generate `sign` I used function from `Cent` module. To see how you can generate API
sign yourself go to chapter "Tokens and signatures".

Also note that in this example we send an array of commands. In this way you can send several
commands to Centrifugo in one request.

There are not so many commands you can call. The main and most useful of them is `publish`.
Lets take a closer look on other available API command methods.

You have `publish`, `unsubscribe`, `presence`, `history`, `disconnect` in your arsenal.

### publish

`publish` allows to send message into channel. `params` for `publish` method must be an
object with two keys: `channel` and `data` which contains valid JSON payload you want to
send into channel.

```javascript
{
    "method": "publish",
    "params": {
        "channel": "CHANNEL NAME",
        "data": {
            "input": "hello"
        }
    }
}
```

Starting with *version 0.2.0* there is an option to include `client` ID into publish API command:

```javascript
{
    "method": "publish",
    "params": {
        "channel": "CHANNEL NAME",
        "data": {
            "input": "hello"
        },
        "client": "long-unique-client-id"
    }
}
```

In most cases this is a `client` ID that initiated this message. This `client` will
be then added on top level of published message.


### unsubscribe

`unsubscribe` allows to unsubscribe user from channel. `params` is an objects with two
keys: `channel` and `user` (user ID you want to unsubscribe)

```javascript
{
    "method": "unsubscribe",
    "params": {
        "channel": "CHANNEL NAME",
        "user": "USER ID"
    }
}
```

### disconnect

`disconnect` allows to disconnect user by its ID. `params` in an object with `user` key.

```javascript
{
    "method": "disconnect",
    "params": {
        "user": "USER ID"
    }
}
```

### presence

`presence` allows to get channel presence information (all clients currently subscribed on
this channel). `params` is an object with `channel` key.

```javascript
{
    "method": "presence",
    "params": {
        "channel": "CHANNEL NAME"
    }
}
```

### history

`history` allows to get channel history information (list of last messages sent into channel).
`params` is an object with `channel` key.

```javascript
{
    "method": "history",
    "params": {
        "channel": "CHANNEL NAME"
    }
}
```
