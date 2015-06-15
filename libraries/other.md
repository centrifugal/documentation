# Other server API client implementations

There are also third party libraries originally built for Centrifuge - predecessor of Centrifugo.
They are not updated to work with Centrifugo, but can be used as starting point for
your communication code.

* [client for Java](https://github.com/mcoetzee/centrifuge-publisher) by Markus Coetzee

To work with Centrifugo client above must use `sha256` HMAC algorithm instead of `md5` and
use project key instead of project ID (project ID does not exist anymore).
