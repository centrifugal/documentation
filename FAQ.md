# FAQ

Answers on various questions here.

### How many connections can Centrifugo handle?

This depends on many factors. Hardware, amount of messages, channel options enabled,
client distribution over channels. So no certain answer on this question exists. Common
sense, tests and monitoring can help here. Generally we suggest to not put more than 50000
clients on one node.

### Can Centrifugo scale horizontally?

Yes and no. Centrifugo scalability now limited by Redis throughput. Redis is very fast – for example
it can handle more than 100000 PUB/SUB messages per second. This should be OK for most applications
in internet. But if you are using Centrifugo and approaching this limit then it's possible to add
sharding support to balance queries between different Redis instances. Just open an issue on Github
and we will work on this.

### Can I publish new messages from client side?

You can and this is what channel option `publish` for. But when publishing from client side
your application will never see that message, so you can't validate it, can't save it into
database. So the idiomatic Centrifugo usage is send new message to your backend first, do whatever
you want with it and then publish to channel via Centrifugo API so it will be broadcasted to all
connected clients subscribed on this channel.

### How I should organize channel namespaces?

The best practice here is use separate namespace for every real-time feature that needs custom
channel options. It's more flexible and configurable in the end.

### Centrifugo stops accepting new connections, why?

The most popular reason behind this is reaching open file limit. Just make it higher.

### Can I use Centrifugo without reverse-proxy like Nginx before it?

Yes, you can - Go standard library designed to allow this. But proxy before Centrifugo can be very useful
for load balancing clients for example.

### Does Centrifugo work with HTTP/2?

Yes, Centrifugo works with HTTP/2.

### Is there a way to use single connection to Centrifugo from different browser tabs?

If underlying transport is HTTP-based and you use HTTP/2 then this will work automatically. In case
of websocket connection there is a way to do this using SharedWorker object.

### What if I need to send push notifications to mobile or web applications?

Sometimes it's confusing to see a difference between real-time messages and push notifications.
Centrifugo is a real-time messaging server. It can not send push notifications to devices - to Apple
iOS devices via APNS, Android devices via GCM or browsers over Web Push API. This is a goal for
another software. But the reasonable question here is how can I know when I need to send real-time
message to client online or push notification to its device because application closed at client's
device at moment. The solution is pretty simple. You can keep critical notifications for client in
database. And when client read message send ack to your backend marking that notification as read
by client, you save this ack too. Periodically you can check which notifications were sent to clients
but they have not read it (no ack received). For such notification you can send push notification
to its device using your own or another open-source solution.



