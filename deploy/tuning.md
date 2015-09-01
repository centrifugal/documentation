# Tuning operating system

As Centrifugo/Centrifuge deals with lots of persistent connections your operating system must be
ready for it.

First of all you should increase a max number of open files your processes can open.

To get you current open files limit run:

```
ulimit -n
```

The result shows approximately how many clients your server can handle.

See http://docs.basho.com/riak/latest/ops/tuning/open-files-limit/ to know how to increase this number.

If you install Centrifugo using RPM from repo then it automatically sets max open files limit to 32768.

You may also need to increase max open files for Nginx.
