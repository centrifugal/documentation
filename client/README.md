# Javascript client

Now you know how Centrifugo server implemented and how it works it's time to connect to it from browser.

For this purpose Centrifugal offers javascript client with simple API.

The source code of javascript client located in its own [repo on Gihub](https://github.com/centrifugal/centrifuge-js).

Javascript client can connect to the server using 2 ways: using pure Websockets or using
SockJS library to be able to use various available fallback transports if client browser
does not support Websockets.

If you've never heard about SockJS - [follow this link](https://github.com/sockjs/sockjs-client) to
get more information about this beautiful library.