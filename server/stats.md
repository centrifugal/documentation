# Server stats and metrics.

When you call `stats` API command you get something like this in response body:

```javascript
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
            "time_api_mean": 0,
            "time_client_mean": 0,
            "time_api_max": 0,
            "time_client_max": 0,
            "memory_sys": 7444728,
            "cpu_usage": 0
        }
    ],
    "metrics_interval": 60
}
```

By default metrics aggregated over 60 seconds period.

"nodes" is an array of stats information from every Centrifugo node running.

`uid` – unique id of node
`name` – name of node
`num_goroutine` – number of active goroutines (Go language specific)
`num_clients` – number of authorized clients connected
`num_unique_clients` – number of unique (with different user ID) clients connected
`num_channels` – number of active channels (with one or more subscribers)
`started_at` – node start time as UNIX timestamp
`gomaxprocs` – number of CPU Centrifugo process can utilize (Go language specific)
`num_cpu` – total number of cpu on machine
`num_msg_published` – number of messages published
`num_msg_queued` – number of messages put into client queues (including protocol messages, join/leave messages etc)
`num_msg_sent` – number of messages actually sent to client (in normal situation must be equal to `num_msg_queued`)
`num_api_requests` – number of requests to server API
`num_client_requests` – number of requests to client API
`bytes_client_in` – number of bytes coming to client API (bytes sent from clients)
`bytes_client_out` – number of bytes coming out of client API (bytes sent to clients)
`time_api_mean` – mean time of API response in nanoseconds
`time_client_mean` – mean time of client API response in nanoseconds.
`time_api_max` – max time of server API response (nanoseconds)
`time_client_max` – max time of client API response (nanoseconds)
`memory_sys` – memory usage in bytes
`cpu_usage` – cpu usage in percents


