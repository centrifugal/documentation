# Python server API client

Python client to work with server API is called `Cent`. The source code is [here](https://github.com/centrifugal/cent).

It's available on Pypi. To install run:

```bash
pip install cent
```

Cent contains Client class to send messages to Centrifuge from your python-powered backend:

```python
from cent.core import Client

client = Client("http://localhost:8000", "project_secret")

params = {
    "channel": "python",
    "data": "hello world"
}

client.add("publish", params)

result, error = client.send()
```

You can use `add` method to add several messages which will be sent. But up to 100 (can be configured via server
configuration)

In response you'll get something like this in response:

```bash
[{'error': None, 'body': None, 'method': 'publish'}]
```

In case of any error you will get its description.

See section about server API to see all available methods and params for them.

### helper methods for available commands

In example above you saw low-level usage of Cent client. There is a more simple usage.

```python
from cent.core import Client

client = Client("http://localhost:8000", "secret")

error = client.publish("public:chat", {"input": "test"})

error = client.unsubscribe("user_id_here")

error = client.disconnect("user_id_here")

messages, error = client.history("public:chat")

clients, error = client.presence("public:chat")

channels, error = client.channels()

stats, error = client.stats()
```

For `publish`, `disconnect`, `unsubscribe` methods only one parameter returns - an error. If no
error occurred the it will be `None`. Otherwise it will be Exception instance.

For other commands that return data calls to helper methods return 2 values - requested data itself
and error (if command was successful `error` will be `None`).

### cent as console client

Cent can also be used as console client to communicate with server API.

By default Cent uses `.centrc` configuration file from your home directory (``~/.centrc``).

In this file you must write several settings - server address and project secret. And optionally timeout.

Here is an example of Cent config file content:

```bash
[bananas]
address = http://localhost:8000
secret = long-secret-key-for-bananas-project
timeout = 5
```

The most obvious case of using Cent is broadcasting events into channels.

It is easy enough:

```bash
cent bananas publish --params='{"channel": "news", "data": {"title": "World Cup 2018", "text": "some text..."}}'
```

- **cent** is the name of program
- **bananas** is the name of section in configuration file
- **publish** is the method name you want to call
- **--params** is a JSON string with method parameters, in case of publish you should provide channel and data parameters.




