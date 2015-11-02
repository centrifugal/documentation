Scaling Centrifugo nodes
========================

As you can read before – it's possible to run multiple nodes of Centrifugo server
and load balance clients between them. In this chapter I'll show how to do it. We
will start 3 Centrifugo nodes and all those nodes will be connected over Redis.

For this purpose we must use Redis engine described in previous chapter.

First, you should have Redis running. As soon as it's running - we can launch 3
Centrifugo instances. Open your terminal and start first one:

```
centrifugo --config=config.json --port=8000 --engine=redis --redis_host=127.0.0.1 --redis_port=6379
```

If your Redis on the same machine and runs on its default port you can omit `--redis_host`
and `--redis_port` options in command above.

Then open another terminal and launch another Centrifugo instance:

```
centrifugo --config=config.json --port=8001 --engine=redis --redis_host=127.0.0.1 --redis_port=6379
```

Note that we use another port number (8001) as port 8000 already busy by our first Centrifugo instance.
If you are starting Centrifugo instances on different machines then you most probably can use
the same port number for all instances.

And let's start third instance:

```
centrifugo --config=config.json --port=8002 --engine=redis --redis_host=127.0.0.1 --redis_port=6379
```

Now you have 3 Centrifugo instances running on ports 8000, 8001, 8002 and clients can connect to
any of them. You can also send API request to any of those nodes - as all nodes connected over Redis
PUB/SUB message will be delivered to all of them.

To load balance clients between nodes we use Nginx - you can find its configuration here in
documentation. Note that it's important to route clients that use polling SockJS transports
(XHR-polling) to the same node as that node keeps client's session information.