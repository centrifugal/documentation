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
client [for Ruby](../libraries/ruby.md) and [PHP](../libraries/php.md). If you use other language don't worry - I will describe
how to communicate with Centrifugo HTTP API endpoint in this chapter.

Note also that there are 2 API endpoints in Centrifugo - first of them is HTTP API endpoint and
second – Redis Engine API. In this section we will talk about HTTP API mostly. You can find
description of Redis API in `engines` chapter - it has the same commands but simplified format.
So if you decide that HTTP API is too difficult and uncomfortable to use - you can use Redis API
to publish new messages into channels.

Let's start!

Centrifugo API url is `/api/`.

So if your Centrifugo sits on domain `https://centrifuge.example.com` then an API address
will be `https://centrifuge.example.com/api/`.

All you need to do to use HTTP API is to send correctly constructed POST request to this endpoint.

API request is a POST `application/json` request with commands in request body and one additional header `X-API-Sign`.

Body of request is just a JSON representing commands you need to execute. This can be single command
or array of commands. See available commands below.

`X-API-Sign` header is an SHA-256 HMAC string based on Centrifugo secret key and JSON body you want to send.
This header is used by Centrifugo to validate API request. In most situations you can protect Centrifugo
API endpoint with firewall rules and disable sign check using `--insecure_api` option when starting
Centrifugo. In this case you just need to send POST request with commands - no need in addition header.

Command is a JSON object with two properties: `method` and `params`.

`method` is a name of action you want to do.
`params` is an object with method arguments.

For example to send publish command to Centrifugo in Python you construct sth like this for your request body:

```
command = json.dumps({
    "method": "publish",
    "params": {"channel": "news", data:{}}
})
```

If you have not turned of sign check you also need to include properly constructed sign
in `X-API-Sign` header when sending this to `/api/` Centrifugo endpoint.

To send 2 publish commands in one request you need body like this:

```
commands = json.dumps([
    {
        "method": "publish",
        "params": {"channel": "news", data:{"content": "1"}}
    },
    {
        "method": "publish",
        "params": {"channel": "news", data:{"content": "2"}}
    }
])
```

First lets see how to construct such request in Python.

*If Python is your language then you don't have to implement this yourself as
`Cent` python module exists.*

But this example here can be useful for someone who want to implement interaction
with Centrifugo API in language for which we don't have API client yet or you just
want to send requests yourself - this is simple and in most cases you can just go
without using our API library (to not introduce extra dependency in your project for
example).

Let's imagine that your Centrifugo has secret key `secret`.

First, let's see how to send API command using Python library `requests`:

```python
import json
import requests

from cent.core import generate_api_sign


commands = [
    {
        "method": "publish",
        "params": {"channel": "docs", "data": {"content": "1"}}
    }
]
encoded_data = json.dumps(commands)
sign = generate_api_sign("secret", encoded_data)
headers = {'Content-type': 'application/json', 'X-API-Sign': sign}
r = requests.post("https://centrifuge.example.com/api/", data=encoded_data, headers=headers)
print r.json()
```

In code above to generate sign we used function from `Cent` module. To see how you can
generate API sign yourself go to chapter [Tokens and signatures](./tokens_and_signatures.md).

Also note that in this example we send an array of commands. In this way you can send several
commands to Centrifugo in one request.

There are not so many commands you can call. The main and most useful of them is `publish`.
Lets take a closer look on other available API command methods.

You have `publish`, `broadcast`, `unsubscribe`, `presence`, `history`, `disconnect`,
`channels`, `stats`, `node` in your arsenal.

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

Starting with **version 0.2.0** there is an option to include `client` ID into publish API command:

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

Response example:

```javascript
{
    "body": null,
    "error": null,
    "method": "publish"
}
```

### broadcast (new in v1.2.0)

Very similar to `publish` but allows to send the same data into many channels.

```javascript
{
    "method": "broadcast",
    "params": {
        "channels": ["CHANNEL_1", "CHANNEL_2"],
        "data": {
            "input": "hello"
        }
    }
}
```

`client` field to set client ID also supported.

This command will publish data into channels until first error happens. This error then set
as response error and publishing stops. In case of using Redis API queue first error will
be logged.


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

Response example:

```javascript
{
    "body": null,
    "error": null,
    "method": "unsubscribe"
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

Response example:

```javascript
{
    "body": null,
    "error": null,
    "method": "disconnect"
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

Response example:

```javascript
{
    "body": {
        "channel": "$public:chat",
        "data": {
            "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239": {
                "user": "2694",
                "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239",
                "default_info": {
                    "first_name": "Alexandr",
                    "last_name": "Emelin"
                },
                "channel_info": {
                    "channel_extra_info_example": "you can add additional JSON data when authorizing"
                }
            },
            "e5ee0ab0-fde1-4543-6f36-13f2201adeac": {
                "user": "2694",
                "client": "e5ee0ab0-fde1-4543-6f36-13f2201adeac",
                "default_info": {
                    "first_name": "Alexandr",
                    "last_name": "Emelin"
                },
                "channel_info": {
                    "channel_extra_info_example": "you can add additional JSON data when authorizing"
                }
            }
        }
    },
    "error": null,
    "method": "presence"
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

Response example:

```javascript
{
    "body": {
        "channel": "$public:chat",
        "data": [
            {
                "uid": "8c5dca2e-1846-42e4-449e-682f615c4977",
                "timestamp": "1445536974",
                "info": {
                    "user": "2694",
                    "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239",
                    "default_info": {
                        "first_name": "Alexandr",
                        "last_name": "Emelin"
                    },
                    "channel_info": {
                        "channel_extra_info_example": "you can add additional JSON data when authorizing"
                    }
                },
                "channel": "$public:chat",
                "data": {
                    "input": "world"
                },
                "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239"
            },
            {
                "uid": "63ecba35-e9df-4dc6-4b72-a22f9c9f486f",
                "timestamp": "1445536969",
                "info": {
                    "user": "2694",
                    "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239",
                    "default_info": {
                        "first_name": "Alexandr",
                        "last_name": "Emelin"
                    },
                    "channel_info": {
                        "channel_extra_info_example": "you can add additional JSON data when authorizing"
                    }
                },
                "channel": "$public:chat",
                "data": {
                    "input": "hello"
                },
                "client": "a1c2f99d-fdaf-4e00-5f73-fc8a6bb7d239"
            }
        ]
    },
    "error": null,
    "method": "history"
}
```


### channels (Centrifugo >= 0.3.0)

`channels` method allows to get list of active (with one or more subscribers) channels.

```javascript
{
    "method": "channels",
    "params": {}
}
```

Response example:

```javascript
{
    "body": {
        "data": [
            "$public:chat",
            "news",
            "notifications"
        ]
    },
    "error": null,
    "method": "channels"
}
```

### stats (Centrifugo >= 1.0.0)

`stats` method allows to get statistics about running Centrifugo nodes.

```javascript
{
    "method": "stats",
    "params": {}
}
```

Response example:

```javascript
{
    "body": {
        "data": {
            "nodes": [
                {
                    "uid": "6045438c-1b65-4b86-79ee-0c35367f29a9",
                    "name": "MacAir.local_8000",
                    "num_goroutine": 21,
                    "num_clients": 0,
                    "num_unique_clients": 0,
                    "num_channels": 0,
                    "started_at": 1445536564,
                    "gomaxprocs": 1,
                    "num_cpu": 4,
                    "num_msg_published": 0,
                    "num_msg_queued": 0,
                    "num_msg_sent": 0,
                    "num_api_requests": 0,
                    "num_client_requests": 0,
                    "bytes_client_in": 0,
                    "bytes_client_out": 0,
                    "memory_sys": 7444728,
                    "cpu_usage": 0
                }
            ],
            "metrics_interval": 60
        }
    },
    "error": null,
    "method": "stats"
}
```

### node (Centrifugo >= 1.4.0)

`node` method allows to get information about single Centrifugo node. That information
will contain counters without aggregation over minute interval (what `stats` method
does by default). So it can be useful if your metric aggregation system aggregates counters
over time period itself. Also note that to use this method you should send API request
to each Centrifugo node separately - as this method returns current raw statistics about
node. See [issue](https://github.com/centrifugal/centrifugo/issues/68) for motivation
description.

```javascript
{
    "method": "node",
    "params": {}
}
```

Response example:

```
{
    "body": {
        "data":{
            "uid":"c3ceab87-8060-4c25-9cb4-94eb9db7899a",
            "name":"MacAir.local_8000",
            "num_goroutine":14,
            "num_clients":0,
            "num_unique_clients":0,
            "num_channels":0,
            "started_at":1455450238,
            "gomaxprocs":4,
            "num_cpu":4,
            "num_msg_published":0,
            "num_msg_queued":0,
            "num_msg_sent":0,
            "num_api_requests":3,
            "num_client_requests":0,
            "bytes_client_in":0,
            "bytes_client_out":0,
            "memory_sys":0,
            "cpu_usage":0
        }
    },
    "error":null,
    "method":"node"
}
```


Note again that there is existing API clients for Python, Ruby, PHP - so you don't
have to construct these commands manually. If you use another programming languages
look at existing clients to get more help implementing call to HTTP API.
