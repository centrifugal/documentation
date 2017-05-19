# Server API libraries

As you could see in Centrifugal overview there are several officially supported libraries
for communicating with server API at moment.

If you have not found a library for your favorite language – you can go completely
without it. You have an option to publish via Redis Engine or just implement
calls to HTTP API yourself - [see server API description](../server/api.md).

Also note that [Cent](https://github.com/centrifugal/cent) contains everything you
need to communicate with server API - so if you have questions - just look at
its source code as reference - as it is written in Python it's very easy to read
and understand. And there are few lines of code actually.

### Python

Python HTTP API client located [on Github](https://github.com/centrifugal/cent).

It can also be used as command-line tool to send various commands to Centrifugo.

### PHP

We have two actual API clients for PHP.

The first one is HTTP API client created by Dmitriy Soldatenko. You can find it [on Github](https://github.com/sl4mmer/phpcent).
It's simple to use - just follow its README and enjoy communicating with Centrifugo HTTP API.

Another API client developed by [Oleh Ozimok](https://github.com/oleh-ozimok) - [php-centrifugo](https://github.com/oleh-ozimok/php-centrifugo).
It allows to work with Redis Engine API queue. See more detailed description in library README.

### Ruby

Ruby client located [on Github](https://github.com/centrifugal/rubycent).

It's very simple to use - just follow its README and enjoy communicating with API.


### NodeJS

NodeJS client located [on Github](https://github.com/centrifugal/jscent).

Basic example:

```javascript
Client = require("jscent");

var c = new Client({url: "http://localhost:8000", secret: "secret"});

c.publish("$public:chat", {"input": "test"}, function(err, resp){console.log(err, resp)});
```

### Go

Go HTTP API client located [on Github](https://github.com/centrifugal/gocent).


### Java

There is third party libraries originally built for Centrifuge - predecessor of Centrifugo.
They are not updated to work with Centrifugo, but can be used as starting point for
your communication code.

* [API library for Java](https://github.com/mcoetzee/centrifuge-publisher) by Markus Coetzee

To work with Centrifugo client above must use `sha256` HMAC algorithm instead of `md5` and
do not use project ID (project ID does not exist anymore).
