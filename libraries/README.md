# Libraries to communicate with server API

![scheme](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/scheme_api.png)

As you could see in Centrifugal overview there are
two officially supported libraries for communicating with server
API at moment - [for Python](https://github.com/centrifugal/cent) and [for Ruby](https://github.com/centrifugal/centrifuge-ruby).

If you don't find a library for your favorite language – you can go completely
without it. You have an option to publish via Redis Engine or just implement
calls to HTTP API yourself - [see server API description](../server/api.md).

Also note that [Cent](https://github.com/centrifugal/cent) contains everything you
need to communicate with server API - so if you have questions - just look at
its source code as reference - as it is written in Python it's very easy to read
and understand. And there are few lines of code actually.
