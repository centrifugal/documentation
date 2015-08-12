# Migrating from Centrifuge to Centrifugo

Centrifugo server has several advantages over Centrifuge (original server in Python):

* performance - thanks to Go language
* single binary file as a release - just download and use Centrifugo
* runs on several cores
* better configuration
* supports handling SIGHUP signal to reload configuration

It's too hard to support both versions so in future only Centrifugo will get new features
in new releases.

Here I'll describe how to migrate to Centrifugo from Centrifuge 0.8.0.

All differences in two command-line option names that changed:

* `log_file` instead of `log_file_prefix`
* `log_level` instead of `logging`

And two configuration file specific options:

* `web_password` instead of `password`
* `web_secret` instead of `cookie_secret`

Redis list key for redis api also was changed: `centrifugo.api` instead `centrifuge.api`

And that's all.
