# Admin web interface

Admin web interface located in its [own repo](https://github.com/centrifugal/web). This
is ReactJS based single-page application. But Centrifugo server comes with this interface builtin.

[See demo on Heroku](https://centrifugo.herokuapp.com) (password `demo`) to see it in action.

![Admin web interface](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/web.gif)

It can:

* show current server general information and statistics from server nodes.
* monitor messages published in channels in real-time (`watch` option for channel must be turned on).
* call `publish`, `unsubscribe`, `disconnect`, `history`, `presence`, `channels`, `stats` server API commands. For
    `publish` command Ace JSON editor helps to write JSON to send into channel.

To enable web you must run `centrifugo` with `--web` flag.

```
centrifugo --config=config.json --admin --web
```

`--admin` enables admin websocket endpoint web interface uses.

`--web` tells Centrifugo that it must serve embedded web interface.

**Note, that you can use only `--web` option to enable both web interface and admin websocket endpoint.
This is because web interface can't work without admin websocket.**

Also you must additionally set 2 options in config: `admin_password` and `admin_secret`.

`config.json`

```json
{
    ...,
    "admin_password": "strong_password_to_log_in",
    "admin_secret": "strong_secret_key_to_sign_authorization_token"
}
```

* `admin_password` – this is a password to log into admin web interface
* `admin_secret` - this is a secret key to sign authorization token used to call admin API endpoints.

Make both strong and keep in secret.

If you don't want to use embedded web interface you can specify path to your own web interface directory:

```
centrifugo --config=config.json --admin --web --web_path=/path/to/web/app
```

This can be useful if you want to modify official web interface in some way.

There is also an option to run Centrifugo in insecure admin mode (new in v1.3.0) - in this case you
don't need to set `admin_password` and `admin_secret` in config - if you use web interface you will
be logged in automatically without any password. Note that this is only for development or if you
protected admin websocket endpoint and web interface with firewall rules in production. To start
Centrifugo with web interface in insecure admin mode run:

```
centrifugo --config=config.json --admin --insecure_admin --web
```
