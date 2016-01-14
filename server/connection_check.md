# Connection check

When client connects to Centrifugo with proper connection credentials then this connection
can live forever. This means that even if you banned this user in your web application
he will be able to read messages from channels he already subscribed to. This is not
what we want in some cases.

Project has special option: `connection_lifetime`. Connection lifetime is `0` by default
i.e. by default connection check mechanism is off.

When connection lifetime is set to value greater than 0 then this is a time in seconds how
long connection will be valid after successful connect. When connection lifetime expires
javascript browser client will make an AJAX POST request to your web application. By default
this request goes to `/centrifuge/refresh/` url endpoint. You can change it using javascript
client configuration option `refreshEndpoint`. In response your server must return JSON with
connection credentials. For example in python:

```python
    to_return = {
        'user': "USER ID,
        'timestamp': "CURRENT TIMESTAMP AS INTEGER",
        'info': "ADDITIONAL CONNECTION INFO",
        'token': "TOKEN BASED ON PARAMS ABOVE",
    }
    return json.dumps(to_return)
```

You must just return the same connection credentials for `user` when rendering page
initially. But with current `timestamp`. Javascript client will then send them to
Centrifugo server and connection will be refreshed for a connection lifetime period.

If you don't want to refresh connection for this user - just return 403 Forbidden
on refresh request to your web application backend.