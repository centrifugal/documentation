# Engines

Engine in Centrifugo is responsible for how to publish message, handle subscriptions, save
or retrieve presence and history data.

By default Centrifugo uses Memory engine. There is also Redis engine available.

The obvious difference between them - with Memory engine you can start only one
node of Centrifugo, while Redis engine allows to run several nodes on different
machines and they will be connected via Redis, will know about each other due to
Redis and will also keep history and presence data in Redis instead of Centrifugo
node process memory.

To set engine you can use `engine` configuration option. Available values are
`memory` and `redis`. Default value is `memory`.

So to work with Redis engine:

```
centrifugo --config=config.json --engine=redis
```

### Memory engine

Supports only one node. Nice choice to start with. Supports all features keeping
everything in Centrifugo node process memory. You don't need to install Redis when
using this engine.


### Redis engine

Allows scaling Centrifuge running multiple nodes on different machines.
Keeps presence and history data in Redis, uses redis PUB/SUB for internal nodes communication.
Also it allows to call API commands.

How to publish via Redis engine API listener? Start Centrifuge with Redis
engine and ``--redis_api`` option:

```bash
centrifuge --logging=debug --config=config.json --engine=redis --redis_api
```

Then use Redis client for your favorite language, ex. for Python:

```python
import redis
import json

client = redis.Redis()

to_send = {
    "data": [
        {
            "method": "publish",
            "params": {"channel": "$public:chat", "data": {"input": "hello"}}
        },
        {
            "method": "publish",
            "params": {"channel": "events", "data": {"event": "message"}}
        },
    ]
}

client.rpush("centrifugo.api", json.dumps(to_send))
```

You send JSON object wth list of commands in `data`.

Note again - you don't have response here as you are adding commands into Redis queue
and they will be processed as soon as Centrifugo can. If you need to get response - you
should use HTTP API.

`publish` is the most usable API command in Centrifugo and Redis API listener was
implemented with primary goal to reduce HTTP overhead when publishing quickly.
This can also help using Centrifugo with other languages for which we don't
have HTTP API client yet.

Several configuration options related to Redis engine:

Run

```bash
centrifugo -h
```

And you will see the following options among the others:

```bash
    --redis_api=false: enable Redis API listener (Redis engine)
    --redis_db="0": redis database (Redis engine)
    --redis_host="127.0.0.1": redis host (Redis engine)
    --redis_password="": redis auth password (Redis engine)
    --redis_port="6379": redis port (Redis engine)
    --redis_url="": redis connection URL (Redis engine)
```

In next chapter we will see how to start several Centrifugo nodes using Redis engine.