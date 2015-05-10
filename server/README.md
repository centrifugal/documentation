# Server implementation description

Originally Centrifugal organization started from [Centrifuge](https://github.com/centrifugal/centrifuge)
real-time messaging server written in Python. But recently the entire Centrifuge codebase was rewritten
into Go language. So from this moment when we will talk about server we mean [Centrifugo](https://github.com/centrifugal/centrifugo).

The goal of server is to accept client connections from browser (which use pure Websockets or SockJS polyfill library as
protocol for communicating), keep accepted connections, deliver different messages from clients to server
(for example subscription command, presence command etc) and from server to clients (new message in channel,
various command responses etc), handle API requests from your web application backend (mostly publish commands
to send message into channel).

