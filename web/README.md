# Admin web interface

Admin web interface located in its [own repo](https://github.com/centrifugal/centrifuge-web). This
is ReactJS based single-page application.

![Admin web interface](https://raw.githubusercontent.com/centrifugal/documentation/master/assets/images/web.gif)

It can:

* show current server general information and statistics from server nodes.
* monitor messages published in channels in real-time (`watch` option for channel must be turned on).
* call `publish`, `unsubscribe`, `disconnect`, `history` and `presence` server API commands. For
    `publish` command Ace JSON editor helps to write JSON to send into channel.

To work with web interface you must serve its `app` folder with Nginx or directly
by `centrifugo` using `--web` option.

Also you mast additionally set 2 options in config: `password` and `secret`.

`config.json`

```json
{
    ...,
    "password": "strong_password_to_log_in",
    "secret": "strong_secret_key_to_sign_authorization_token"
}
```

* `password` – this is a password to log into admin web interface
* `secret` - this is a secret key to sign authorization token used to call web API endpoints.

Make both strong and keep in secret.