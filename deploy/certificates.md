# TLS certificates

TLS/SSL layer is very important not only for securing your connections but also to increase a
chance to establish Websocket connection. **In most situations you will put TLS termination task
on your reverse proxy/load balancing software such as Nginx**.

There are situations though when you want to serve secure connections by Centrifugo itself.

There are two ways to do this: using TLS certificate `cert` and `key` files that you've got
from your CA provider or using automatic certificate handling via [ACME](https://ietf-wg-acme.github.io/acme/) provider (only
[Let's Encrypt](https://letsencrypt.org/) at this moment).

### Using crt and key files

In first way you already have `cert` and `key` files. For development you can create self-signed
certificate - see [this instruction](https://devcenter.heroku.com/articles/ssl-certificate-self) as
example.

Then to start Centrifugo use the following command:

```
./centrifugo --config=config.json --ssl --ssl_key=server.key --ssl_cert=server.crt
```

Or just use configuration file:

```json
{
  ...
  "ssl": true,
  "ssl_key": "server.key",
  "ssl_cert": "server.crt"
}
```

And run:

```
./centrifugo --config=config.json
```

### Automatic certificates

For automatic certificates from Let's Encrypt add into configuration file:

```
{
  ...
  "ssl_autocert": true,
  "ssl_autocert_host_whitelist": "www.example.com",
  "ssl_autocert_cache_dir": "/tmp/certs",
  "ssl_autocert_email": "user@example.com"
}
```

`ssl_autocert` says Centrifugo that you want automatic certificate handling using ACME provider.

`ssl_autocert_host_whitelist` is a string with your app domain address. This can be comma-separated
list. It's optional but recommended for extra security.

`ssl_autocert_cache_dir` is a path to a folder to cache issued certificate files. This is optional
but will increase performance.

`ssl_autocert_email` is optional - it's an email address ACME provider will send notifications
about problems with your certificates.

When configured correctly and your domain is valid (`localhost` will not work) - certificates
will be retrieved on first request to Centrifugo.
