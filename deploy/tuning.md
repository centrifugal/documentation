# Tuning operating system

As Centrifugo/Centrifuge deals with lots of persistent connections your operating system must be
ready for it.

### open files limit

First of all you should increase a max number of open files your processes can open.

To get you current open files limit run:

```
ulimit -n
```

The result shows approximately how many clients your server can handle.

See http://docs.basho.com/riak/latest/ops/tuning/open-files-limit/ to know how to increase this number.

If you install Centrifugo using RPM from repo then it automatically sets max open files limit to 32768.

You may also need to increase max open files for Nginx.

### lots of sockets in TIME_WAIT state.

Look how many descriptors in TIME_WAIT state.

```
netstat -an |grep TIME_WAIT | grep CENTRIFUGO_PID | wc -l
```

Under load when lots of connections and disconnection happen lots of used socket descriptors can
stay in TIME_WAIT state. Those descriptors can not be reused for a while. So you can get various
errors when using Centrifugo. For example something like `(99: Cannot assign requested address)
while connecting to upstream` in Nginx error log and 502 on client side. In this case there are
several advices that can help.

There is a perfect article about operating system tuning for lots of connections: https://engineering.gosquared.com/optimising-nginx-node-js-and-networking-for-heavy-workloads.

To summarize:

1. Increase local port range
2. Enable tcp_tw_reuse
3. If you are using Nginx set `keepalive` directive in upstream.

```
upstream centrifugo {
    #sticky;
    ip_hash;
    server 127.0.0.1:8000;
    keepalive 512;
}
```
