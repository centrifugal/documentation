# Other server API client implementations

If you implemented library in new language - please write us about it.

### Java

There is third party libraries originally built for Centrifuge - predecessor of Centrifugo.
They are not updated to work with Centrifugo, but can be used as starting point for
your communication code.

* [API library for Java](https://github.com/mcoetzee/centrifuge-publisher) by Markus Coetzee

To work with Centrifugo client above must use `sha256` HMAC algorithm instead of `md5` and
do not use project ID (project ID does not exist anymore).

