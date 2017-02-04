# Server stats and metrics.

When you call `stats` API command you get something like this in response body:

```javascript
"body": {
    "data": {
        "nodes": [
            {
                "uid": "8a7cf241-62c3-4ef5-8a20-83fbadb74c6f",
                "name": "Alexanders-MacBook-Pro.local_8000",
                "started_at": 1480764329,
                "metrics": {
                    "client_api_15_count": 0,
                    "client_api_15_microseconds_50%ile": 0,
                    "client_api_15_microseconds_90%ile": 0,
                    "client_api_15_microseconds_99%ile": 0,
                    "client_api_15_microseconds_99.99%ile": 0,
                    "client_api_15_microseconds_max": 0,
                    "client_api_15_microseconds_mean": 0,
                    "client_api_15_microseconds_min": 0,
                    "client_api_1_count": 0,
                    "client_api_1_microseconds_50%ile": 0,
                    "client_api_1_microseconds_90%ile": 0,
                    "client_api_1_microseconds_99%ile": 0,
                    "client_api_1_microseconds_99.99%ile": 0,
                    "client_api_1_microseconds_max": 0,
                    "client_api_1_microseconds_mean": 0,
                    "client_api_1_microseconds_min": 0,
                    "client_api_num_requests": 0,
                    "client_bytes_in": 0,
                    "client_bytes_out": 0,
                    "client_num_connect": 0,
                    "client_num_msg_published": 0,
                    "client_num_msg_queued": 0,
                    "client_num_msg_sent": 0,
                    "client_num_subscribe": 0,
                    "http_api_15_count": 0,
                    "http_api_15_microseconds_50%ile": 0,
                    "http_api_15_microseconds_90%ile": 0,
                    "http_api_15_microseconds_99%ile": 0,
                    "http_api_15_microseconds_99.99%ile": 0,
                    "http_api_15_microseconds_max": 0,
                    "http_api_15_microseconds_mean": 0,
                    "http_api_15_microseconds_min": 0,
                    "http_api_1_count": 0,
                    "http_api_1_microseconds_50%ile": 0,
                    "http_api_1_microseconds_90%ile": 0,
                    "http_api_1_microseconds_99%ile": 0,
                    "http_api_1_microseconds_99.99%ile": 0,
                    "http_api_1_microseconds_max": 0,
                    "http_api_1_microseconds_mean": 0,
                    "http_api_1_microseconds_min": 0,
                    "http_api_num_requests": 0,
                    "http_raw_ws_num_requests": 0,
                    "http_sockjs_num_requests": 0,
                    "node_cpu_usage": 0,
                    "node_memory_sys": 10524920,
                    "node_num_add_client_conn": 0,
                    "node_num_add_client_sub": 0,
                    "node_num_add_presence": 0,
                    "node_num_admin_msg_published": 0,
                    "node_num_admin_msg_received": 0,
                    "node_num_channels": 0,
                    "node_num_client_msg_published": 0,
                    "node_num_client_msg_received": 0,
                    "node_num_clients": 0,
                    "node_num_control_msg_published": 21,
                    "node_num_control_msg_received": 21,
                    "node_num_goroutine": 12,
                    "node_num_history": 0,
                    "node_num_join_msg_published": 0,
                    "node_num_join_msg_received": 0,
                    "node_num_last_message_id": 0,
                    "node_num_leave_msg_published": 0,
                    "node_num_leave_msg_received": 0,
                    "node_num_presence": 0,
                    "node_num_remove_client_conn": 0,
                    "node_num_remove_client_sub": 0,
                    "node_num_remove_presence": 0,
                    "node_num_unique_clients": 0,
                    "node_uptime_seconds": 120,
                }
            }
        ],
        "metrics_interval": 60
    }
}
```

By default Centrifugo aggregates metrics over 60 seconds period. You can change this begaviour using `node_metrics_interval` configuration option.

Let's look what else you can see in `stats` response body.

"nodes" is an array of stats information from every Centrifugo node running.

Inside every node:

`uid` – unique id of node

`name` – name of node

`started_at` – node start time as UNIX timestamp

`metrics` is a map of metric values (keys always `string`, values always `integer`) - note that this is a snapshot, it only changes once in `node_metrics_interval` period.


Let's describe some metrics in detail:

`node_memory_sys` – node memory usage in bytes

`node_cpu_usage` – node cpu usage in percents

`node_num_goroutine` – number of active goroutines (Go language specific)

`node_num_clients` – number of connected authorized clients

`node_num_unique_clients` – number of unique (with different user ID) clients connected

`node_num_channels` – number of active channels (with one or more subscribers)

`node_num_client_msg_published` – number of messages published

`node_uptime_seconds` – seconds passed from time when node started.

`http_api_num_requests` – number of requests to server HTTP API

`client_api_num_requests` – number of requests to client API

`client_bytes_in` – number of bytes coming to client API (bytes sent from clients)

`client_bytes_out` – number of bytes coming out of client API (bytes sent to clients)

`client_num_msg_queued` – number of messages put into client queues (including protocol messages, join/leave messages etc)

`client_num_msg_sent` – number of messages actually sent to client (in normal situation must be equal to `num_msg_queued`)

There are also HDR histogram metric values: for HTTP API and for client API. They are collected over 1 and 15 interval buckets.

Buckets rotated every `node_metric_interval` seconds. So By default you see values over 1 minute and 15 minute.
