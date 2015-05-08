# Structure

Structure contains an array of projects. Project is an object with 2 required fields: `name` and `secret`.

So the simplest structure looks like this:

```javascript
{
    "structure": [
        {
            "name": "bananas",
            "secret": "very-long-secret-key-for-bananas-project"
        }
    ]
}
```

* `name` – unique name of project (must be in ascii). Used as project key when sending API requests and must be provided by clients when they want to connect to Centrifugo from browser.
* `secret` – secret key for project. The only two who should know this secret key is Centrifugo itself and your web application backend. It used to generate client connection tokens (more about them later), sign API requests and sign private channel subscription requests.

The next option is `connection_lifetime`:

* `connection_lifetime` – time in seconds for client connection to expire. By default it equals 0 - this means that connection can live forever and will not expire. See more about connection expiration mechanism in special chapter.

All other project options related to channels. Channel is an entity to which clients can subscribe to receive messages
published into that channel. Channel is just a string - but several symbols has special meaning - see next section to get
more information about channels. But now you should only understand that next project options will affect channel
settings for this project:


