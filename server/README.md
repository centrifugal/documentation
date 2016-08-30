# Server implementation description

Originally Centrifugal organization started from [Centrifuge](https://github.com/centrifugal/centrifuge)
real-time messaging server written in Python. But then the entire Centrifuge codebase was rewritten
into Go language. So from this moment when we will talk about server we mean [Centrifugo](https://github.com/centrifugal/centrifugo).

The goal of server is to accept client connections from application (web app, mobile app,
desktop app). Connections can use pure [Websockets](https://developer.mozilla.org/en/docs/WebSockets)
protocol or [SockJS](https://github.com/sockjs/sockjs-client) polyfill library protocol. Centrifugo
keeps accepted connections, deliver different messages from clients to server (for example subscription
command, presence command etc) and from server to clients (new message in channel, various command
responses etc), handle API requests from your web application backend (mostly publish commands to
send new message into channel).
