# Client libraries to communicate with Centrifugo

This chapter is about client libraries. Centrifugo originally created to work
with browser clients. But Websocket protocol built on top of convenient TCP and very
popular so it's very easy to find Websocket client for almost every existing language
and write client for Centrifugo on top of it.

The central part of this section is javascript browser client - `centrifuge-js``.

![scheme](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/scheme_client.png)

Go client is in many aspects similar to `centrifuge-js` but allows to work with Centrifugo
from non-browser environment.
