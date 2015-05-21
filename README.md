Welcome to Centrifugal docs
===========================

![Centrifugal intro](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/intro.png)

Centrifugal organization is a set of tools to add real-time features on your web site.

It brings together several repositories linked by a common purpose – give you a complete
and ready to use solution when you want to add real-time events into your web application.

Chats, real-time graphs and notifications, various counters and even games can be built
using our instruments – real-time messaging server, javascript browser client and client
libraries for your favorite language.

Our real-time server easily integrates with your existing web site – you don't need
to change your project architecture and philosophy to get real-time events.

On client side users of your web application communicate with real-time server using our
javascript client over Websockets or [SockJS](https://github.com/sockjs/sockjs-client)
library protocol. SockJS fallback transports provide real-time messaging support even
in old (like IE 7) or mobile browsers.

There are tons of examples in internet about how to add real-time events on your site.
But very few of them provide complete, full-featured and ready to deploy solution.
Centrifugal organization aims to give such a solution with simplicity in mind.

### Centrifugal projects

Let's see which projects Centrifugal has:

* [centrifugo](https://github.com/centrifugal/centrifugo) - real-time messaging server
    written in Go language. This is a successor of
    [Centrifuge](https://github.com/centrifugal/centrifugo) – original server implementation
    written in Python.
* [centrifuge-js](https://github.com/centrifugal/centrifuge-js) - Javascript client to
    connect to messaging server from web browser.
* [web](https://github.com/centrifugal/web) - admin web interface for Centrifugo.
    Built on ReactJS.
* [cent](https://github.com/centrifugal/cent) - Python tools to communicate with Centrifugo API.
* [adjacent](https://github.com/centrifugal/adjacent) - a small wrapper over Cent to
    simplify real-time server integration with Django framework.
* [centrifuge-ruby](https://github.com/centrifugal/centrifuge-ruby) - Ruby gem to communicate
    with Centrifugo API.

### Examples

All examples can be found [in repo on Github](https://github.com/centrifugal/examples).

At moment we have the following examples:

* django – example shows how to integrate Django application with Centrifugal stack
* tornado application – shows some general aspects of Centrifugal stack - token generation, private channel signing.
* insecure - example shows how to use Centrifugo running in insecure mode without any web application backend.
* jsfiddle - [simplified chat example on jsfiddle](http://jsfiddle.net/FZambia/yG7Uw/) with predefined user ID, timestamp
    and token which uses [Centrifugo demo instance on Heroku](https://centrifugo.herokuapp.com)


{% include "SUMMARY.md" %}