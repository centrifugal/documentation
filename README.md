## Centrifugo real-time messaging server and its friends

![Centrifugal intro](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/intro.png)

Centrifugal [organization](https://github.com/centrifugal) provides a set of tools to add real-time features to your web/mobile/desktop application. It brings together several repositories linked by a common purpose – give you a complete and ready to use solution when you want to add real-time events into your application.

Chats, real-time charts, notifications, various counters and even games can be built using our instruments – [real-time messaging server](server/README.md), [javascript browser client](client/README.md) and [client API libraries](libraries/README.md) for your favorite language.

Centrifugo server easily integrates with your existing application – no need to change your project architecture and philosophy to get real-time events. Centrifugo server is language agnostic – its API can be used from any programming language. For some popular programming languages (see below) we provide helper libraries, but protocol is open and simple - so it's possible to create your own wrapper.

On client side users of your application communicate with real-time Centrifugo over Websocket or [SockJS](https://github.com/sockjs/sockjs-client) library protocol. SockJS fallback transports provide real-time messaging support even in old (like IE 8) or mobile browsers.

### Our projects

Let's see which projects Centrifugal organization has:

* [centrifugo](https://github.com/centrifugal/centrifugo) - real-time messaging server
    written in Go language
* [centrifuge-js](https://github.com/centrifugal/centrifuge-js) - Javascript client to
    connect to messaging server from web browser.
* [centrifuge-android](https://github.com/centrifugal/centrifuge-android) - Java library to communicate
    with Centrifugo client API over Websockets from Android devices.
* [centrifuge-ios](https://github.com/centrifugal/centrifuge-ios) - Swift library to communicate
    with Centrifugo client API over Websockets from iOS devices.
* [centrifuge-go](https://github.com/centrifugal/centrifuge-go) - Go library to communicate
    with Centrifugo client API over Websockets from Go apps.
* [cent](https://github.com/centrifugal/cent) - Python tools to communicate with Centrifugo API.
* [adjacent](https://github.com/centrifugal/adjacent) - a small wrapper over Cent to
    simplify real-time server integration with Django framework.
* [rubycent](https://github.com/centrifugal/rubycent) - Ruby gem to communicate
    with Centrifugo server API.
* [phpcent](https://github.com/centrifugal/phpcent) - PHP client to communicate
    with Centrifugo server API.
* [gocent](https://github.com/centrifugal/gocent) - Go client to communicate
    with Centrifugo server API.
* [jscent](https://github.com/centrifugal/jscent) - NodeJS client to communicate
    with Centrifugo server API.
* [web](https://github.com/centrifugal/web) - admin web interface for Centrifugo.
    Built on ReactJS.

### Demo

We maintain actual [demo of Centrifugo server instance](https://centrifugo.herokuapp.com) on Heroku (password `demo`).

You can use it to discover Centrifugo without installing it on your computer.

There are 3 endpoints of demo available:

* wss://centrifugo.herokuapp.com/connection/websocket - raw Websocket endpoint
* https://centrifugo.herokuapp.com/connection - SockJS endpoint
* https://centrifugo.herokuapp.com/api/ - HTTP API endpoint

So you can use our client and API libraries to communicate with this demo.

### Examples

Examples can be found [in repo on Github](https://github.com/centrifugal/examples).

At moment we have the following examples:

* django – example shows how to integrate Django application with Centrifugal stack
* tornado application – shows some general aspects of Centrifugal stack - token generation, private channel signing.
* WebRTC chat - shows how to use Centrifugo as WebRTC signaling server to create peer-to-peer communication.
* insecure – example shows how to use Centrifugo running in insecure mode without any web application backend.
* jsfiddle – [simplified chat example on jsfiddle](http://jsfiddle.net/FZambia/yG7Uw/) with predefined user ID, timestamp and token which uses Centrifugo [demo](https://centrifugo.herokuapp.com) instance on Heroku

### Simplified scheme

![scheme](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/scheme.png)

{% include "SUMMARY.md" %}
