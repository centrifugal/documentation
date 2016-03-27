# Message delivery model

The model of message delivery of Centrifugo server is `at most once`.

This means that message you send to Centrifugo can be theoretically lost while moving towards
your clients. Centrifugo tries to do a best effort to prevent message losses but you should be
aware of this fact. Your application should tolerate this. Centrifugo has an option to
automatically recover messages that have been lost because of short network disconnections. But
there are cases when Centrifugo can't guarantee message delivery. We also recommend to model your
applications in a way that users don't notice when message have been lost. For example if your user
posts a new comment over AJAX call to your application backend - you should not rely only on
Centrifugo to get new comment form and display it - you should return new comment data in AJAX call
response and render it. Be careful to not draw comments twice in this case.

Message order in channels guaranteed to be the same while you publish messages into channel one after
another or publish them in one request (array of published messages). If you do parallel
publishes into the same channel then Centrifugo can't strongly guarantee message order.
