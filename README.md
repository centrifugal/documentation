Welcome to Centrifugal docs
===========================

![Centrifugal intro](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/intro.png)

Centrifugal organization is a set of tools to add real-time features on your web site.

It brings together several repositories linked by a common purpose – give you a complete
and ready to use solution when you want to add real-time events into your web application.
Chats, real-time graphs, notifications, counters and even games can be built using our
instruments – real-time messaging server, javascript browser client and client libraries
for your favorite language.

Our real-time server easily integrates with your existing web site – you don't need
to change your project architecture and philosophy to get real-time events.

There are tons of examples in internet about how to add real-time events on your site. But
very few of them provide complete, scalable, full-featured, ready to deploy solution.
Centrifugal organization aims to give such a solution with simplicity in mind.

### Centrifugal projects

Let's see which projects Centrifugal has:

* [centrifuge](https://github.com/centrifugal/centrifuge) - real-time messaging server written in Python. **This is deprecated - see Centrifugo below as brand-new replacement.**
* [centrifugo](https://github.com/centrifugal/centrifugo) - real-time messaging server written in Go. This is a brand-new almost drop-in replacement for Centrifuge. It has several advantages over Python version and hopefully will be the only supported server implementation in future.
* [centrifuge-web](https://github.com/centrifugal/centrifuge-web) - admin web interface for Centrifuge/Centrifugo. Built on ReactJS.
* [centrifuge-js](https://github.com/centrifugal/centrifuge-js) - Javascript client to connect to messaging server from web browser.
* [cent](https://github.com/centrifugal/cent) - Python tools to communicate with Centrifuge/Centrifugo.
* [adjacent](https://github.com/centrifugal/adjacent) - a small wrapper over Cent to simplify Centrifuge integration with Django framework.
* [centrifuge-ruby](https://github.com/centrifugal/centrifuge-ruby) - Ruby gem to communicate with Centrifuge/Centrifugo

### Examples

At moment all examples [can be found in Centrifuge server repo](https://github.com/centrifugal/centrifuge/tree/master/examples).
All examples work well with Centrifugo server.

{% include "SUMMARY.md" %}