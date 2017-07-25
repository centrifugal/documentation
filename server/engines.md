# Engines

* [Memory engine](#memory-engine)
* [Redis engine](#redis-engine)

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

Allows scaling Centrifugo nodes to different machines. Nodes will use Redis
as message broker. Redis engine keeps presence and history data in Redis, uses Redis
PUB/SUB for internal nodes communication. Also it allows to enqueue API commands.

![scheme](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/scheme_redis.png)

How to publish via Redis engine API listener? Start Centrifugo with Redis engine and
``--redis_api`` option:

```bash
centrifugo --config=config.json --engine=redis --redis_api
```

Note, that starting from Centrifugo 1.6.0 Redis API message format changed. You can find old
format description [here](https://github.com/centrifugal/documentation/blob/2eadd7d3f9991c46c463aff4126f2ea37b17bfad/server/engines.md#redis-engine). Old format deprecated and will be removed in future releases.

Use Redis client for your favorite language, ex. for Python:

```python
import redis
import json

client = redis.Redis()

command = {
    "method": "publish",
    "params": {
        "channel": "events",
        "data": {"event": "message"}
    }
}

client.rpush("centrifugo.api", json.dumps(command))
```

[RPUSH](https://redis.io/commands/rpush) Redis command allows to push several messages
into queue in one request.

Note that we RPUSH messages into `centrifugo.api` - this is a default name of API queue
Centrifugo watches for. Actually this is just a LIST data structure in Redis.

Again - you don't have response here as you are adding commands into Redis queue
and they will be processed as soon as Centrifugo can. If you need to get response - you
should use HTTP API.

`publish` is the most usable API command in Centrifugo and Redis API listener was implemented
with primary goal to reduce HTTP overhead when publishing quickly. This can also help using
Centrifugo with other languages for which we don't have HTTP API client yet.

Several configuration options related to Redis engine:

Run

```bash
centrifugo -h
```

And you will see the following options among the others:

```bash
    --redis_api                  enable Redis API listener (Redis engine)
    --redis_api_num_shards int   Number of shards for redis API queue (Redis engine)
    --redis_db string            redis database (Redis engine) (default "0")
    --redis_host string          redis host (Redis engine) (default "127.0.0.1")
    --redis_master_name string   Name of Redis master Sentinel monitors (Redis engine)
    --redis_password string      redis auth password (Redis engine)
    --redis_pool int             Redis pool size (Redis engine) (default 256)
    --redis_port string          redis port (Redis engine) (default "6379")
    --redis_sentinels string     Comma separated list of Sentinels (Redis engine)
    --redis_url string           redis connection URL in format redis://:password@hostname:port/db (Redis engine)
```

Most of these options are clear – `--redis_host`, `--redis_port`, `--redis_password`, `--redis_db`, `--redis_pool`

`--redis_url` allows to set Redis connection parameters in a form of URL in format `redis://:password@hostname:port/db_number`.

When `--redis_url` set Centrifugo will use URL instead of values provided in `--redis_host`,
`--redis_port`, `--redis_password`, `--redis_db` options.

The most advanced option here is `--redis_api_num_shards`. It's new in v1.3.0. This option must be
used in conjunction with `--redis_api`, i.e. it makes sense only when Redis API enabled. It creates
up to N additional shard queues that Centrifugo instance will listen to new API commands.

For example if you set `--redis_api_num_shards` to five then Centrifugo will listen to following
queues in Redis:

```
centrifugo.api.0
centrifugo.api.1
centrifugo.api.2
centrifugo.api.3
centrifugo.api.4
```

You can push new commands to any of these queues and commands will be received by Centrifugo instance
and processed. Why do we need this?

As we described above when using `--redis_api` you can publish new messages using RPUSH command
into `centrifugo.api` queue in Redis. This is OK until you have small amount of new messages that
must be published. But what if you have thousands of new messages per second? The solution is to
use these shard queues. You distribute messages over those queues and this allows to increase
throughput of Centrifugo. Note that you must decide on your client side to which queue you are going
to push message. To keep message order in channels it's important to push messages belonging to the
same channel into the same queue. This can be achieved using something like `crc16(CHANNEL_NAME) mod N`
function where N is number of shard queues (i.e. ``--redis_api_num_shards`` option).

In next chapter we will see how to start several Centrifugo nodes using Redis engine.
