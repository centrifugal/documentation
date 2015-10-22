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
centrifugo --config=config.json --web
```

Also you must additionally set 2 options in config: `web_password` and `web_secret`.

`config.json`

```json
{
    ...,
    "web_password": "strong_password_to_log_in",
    "web_secret": "strong_secret_key_to_sign_authorization_token"
}
```

* `web_password` – this is a password to log into admin web interface
* `web_secret` - this is a secret key to sign authorization token used to call web API endpoints.

Make both strong and keep in secret.

If you don't want to use embedded web interface you can specify path to your own web interface directory:

```
centrifugo --config=config.json --web --web_path=/path/to/web/app
```

This can be useful if you want to modify official web interface in some way.