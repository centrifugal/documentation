# Tokens and signatures

Centrifugo uses HMAC algorithm to create connection token and to sign various
data when communicating with Centrifugo via server or client API.

In this chapter we will see how to create tokens and signatures for different
actions. If you use Python all functions available in `Cent` library and you
don't need to implement them. This chapter can be useful for developers building
their own library (in other language for example) to communicate with Centrifugo.

Lets start with connection token.

### Connection token

When client connects to Centrifuge from browser it should provide several connection
parameters: "project", "user", "timestamp", "info" and "token".

We discussed the meaning of parameters in other chapters - here we will see
how to generate a proper token for them.

Let's look at Python code for this:

```python
import six
import hmac
from hashlib import sha256

def generate_token(secret_key, project_key, user, timestamp, info=None):
    if info is None:
        info = json.dumps({})
    sign = hmac.new(six.b(secret_key), digestmod=sha256)
    sign.update(six.b(project_key))
    sign.update(six.b(user))
    sign.update(six.b(timestamp))
    sign.update(six.b(info))
    token = sign.hexdigest()
    return token
```

We initialize HMAC with project secret key and ``sha256`` digest mode and then update
it with project_key, user ID, timestamp and info. Info is an optional arguments and if
no info provided empty object is used by default.


### Private channel subscription sign

When client wants to subscribe on private channel Centrifuge js client sends AJAX POST
request to your web application. This request contains `client` ID string and one or multiple
private `channels`. In response you should return an object where channels are keys.

For example you received request with channels `$one` and `$two`. Then you should return
JSON with something like this in response:

```python
{
    "$one": {
        "info": "{}",
        "sign": "ssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss"
    },
    "$two": {
        "info": "{}",
        "sign": "ssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssssss"
    }
}
```

Where `info` is additional information about connection for this channel and `sign` is
properly constructed HMAC based on client ID, channel name and info. Lets look at Python code
generating this sign:

```python
import six
import hmac
from hashlib import sha256

def generate_channel_sign(secret_key, client, channel, info=None):
    if info is None:
        info = json.dumps({})
    auth = hmac.new(six.b(secret_key), digestmod=sha256)
    auth.update(six.b(str(client)))
    auth.update(six.b(str(channel)))
    auth.update(six.b(info))
    return auth.hexdigest()
```

Not so different from generating token. Note that as with token - info is already JSON
encoded string.


### API request sign

When you use Centrifugo server API you should also provide sign in each request.

Again, Python code for this:

```python
import six
import hmac
from hashlib import sha256

def generate_api_sign(self, secret_key, project_key, encoded_data):
    sign = hmac.new(six.b(secret_key), digestmod=sha256)
    sign.update(six.b(project_key))
    sign.update(six.b(encoded_data))
    return sign.hexdigest()
```

`encoded_data` is already a JSON string with your API commands. See available commands
in server API chapter.
