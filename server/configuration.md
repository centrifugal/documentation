# Configuration overview

Centrifugo expects JSON, TOML or YAML formats as format of configuration file.
Thanks to brilliant Go library for application configuration - [viper](https://github.com/spf13/viper).

But first let's inspect all available command-line options:

```bash
centrifugo -h
```

You should see something like this as output:

```
Centrifugo. Real-time messaging (Websockets or SockJS) server in Go.

Usage: 
   [flags]
   [command]
Available Commands: 
  version     Centrifugo version number
  checkconfig Check configuration file
  genconfig   Generate simple configuration file to start with
  help        Help about any command

Flags:
  -a, --address string             address to listen on
      --admin                      enable admin socket
      --admin_port string          port to bind admin endpoints to (optional)
      --api_port string            port to bind api endpoints to (optional)
  -c, --config string              path to config file (default "config.json")
  -d, --debug                      enable debug mode
  -e, --engine string              engine to use: memory or redis (default "memory")
      --insecure                   start in insecure client mode
      --insecure_admin             use insecure admin mode – no auth required for admin socket
      --insecure_api               use insecure API mode
      --log_file string            optional log file - if not specified logs go to STDOUT
      --log_level string           set the log level: trace, debug, info, error, critical, fatal or none (default "info")
  -n, --name string                unique node name
      --pid_file string            optional path to create PID file
  -p, --port string                port to bind HTTP server to (default "8000")
      --redis_api                  enable Redis API listener (Redis engine)
      --redis_api_num_shards int   Number of shards for redis API queue (Redis engine)
      --redis_db string            redis database (Redis engine) (default "0")
      --redis_host string          redis host (Redis engine) (default "127.0.0.1")
      --redis_master_name string   Name of Redis master Sentinel monitors (Redis engine)
      --redis_password string      redis auth password (Redis engine)
      --redis_pool int             Redis pool size (Redis engine) (default 256)
      --redis_port string          redis port (Redis engine) (default "6379")
      --redis_sentinels string     Comma separated list of Sentinels (Redis engine)
      --redis_url string           redis connection URL in format redis://:password@hostname:port/db (Redis engine)
      --ssl                        accept SSL connections. This requires an X509 certificate and a key file
      --ssl_cert string            path to an X509 certificate file
      --ssl_key string             path to an X509 certificate key
  -w, --web                        serve admin web interface application (warning: automatically enables admin socket)
      --web_path string            optional path to custom web interface application

Global Flags:
  -h, --help   help for 

Use " help [command]" for more information about a command.
```

### version

To show version and exit run:

```
centrifugo version
```

### configuration JSON file example

But the subject of this section - configuration file. As was mentioned earlier it must be a file with valid JSON.

Let's look at configuration file example I personally use while developing Centrifugo:

```javascript
{
  "secret": "secret",
  "namespaces": [
    {
      "name": "public",
      "publish": true,
      "watch": true,
      "presence": true,
      "join_leave": true,
      "history_size": 10,
      "history_lifetime": 30
    }
  ],
  "log_level": "debug"
}
```

Only **secret** options is required.

You will know about other options such as `namespaces` in next sections.

So the **minimal configuration file required** is:

```javascript
{
  "secret": "secret"
}
```

But use strong secret in production!

### TOML

Centrifugo also supports TOML format for configuration file:

```
centrifugo --config=config.toml
```

Where `config.toml` contains:

```
log_level = "debug"

secret = "secret"

[[namespaces]]
    name = "public"
    publish = true
    watch = true
    presence = true
    join_leave = true
    history_size = 10
    history_lifetime = 30
```

I.e. the same configuration as JSON file above.

### YAML

And YAML config also supported. `config.yaml`:

```
log_level: debug

secret: secret
namespaces:
  - name: public
    publish: true
    watch: true
    presence: true
    join_leave: true
    history_size: 10
    history_lifetime: 30
```

With YAML remember to use spaces, not tabs when writing configuration file

### multiple projects

Since Centrifugo 1.0.0 multiple projects not supported.

### checkconfig

Centrifugo has special command to check configuration file `checkconfig`:

```bash
centrifugo checkconfig --config=config.json
```

If any errors found during validation – program will exit with error message and exit status 1.

### genconfig

Another command is `genconfig`:

```
centrifugo genconfig -c config.json
```

It will generate the simplest configuration file for you automatically.

### important command-line options

In next section we will talk about project settings in detail. But before jumping to it
let's describe some of the most important options you can configure when running Centrifugo:

* `--address` – bind your Centrifugo to specific interface address (by default `""`)
* `--port` – port to bind Centrifugo to (by default `8000`)
* `--engine` – engine to use - `memory` or `redis` (by default `memory`). Read more about engines in next sections.
* `--web` – path to directory of admin web interface application to serve
* `--name` – give Centrifugo server node a name – this os optional as by default Centrifugo will use hostname
    and port number to construct node name.

There are more command line options – we will talk about some of them later. Note that all command-line options can
be set via configuration file, but command-line options will be more valuable when set than configuration file's options.
See description of [viper](https://github.com/spf13/viper) – to see more details about configuration options priority.
