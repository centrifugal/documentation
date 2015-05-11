# Configuration overview

Centrifugo expects JSON, TOML or YAML formats as format of configuration file.
Thanks to brilliant Go library for application configuration - [viper](https://github.com/spf13/viper).

But first let's inspect all available command-line options:

```bash
centrifugo -h
```

You should see something like this as output:

```bash
Centrifuge + GO = Centrifugo – harder, better, faster, stronger

Usage:
   [flags]
   [command]

Available Commands:
  version     Centrifugo version number
  help        Help about any command

Flags:
  -a, --address="localhost": address
  -c, --config="config.json": path to config file
  -d, --debug=false: debug mode - please, do not use it in production
  -e, --engine="memory": engine to use: memory or redis
      --insecure=false: start in insecure mode
      --log_file="": optional log file - if not specified all logs go to STDOUT
      --log_level="info": set the log level: debug, info, error, critical, fatal or none
  -n, --name="": unique node name
  -p, --port="8000": port
      --redis_api=false: enable Redis API listener (Redis engine)
      --redis_db="0": redis database (Redis engine)
      --redis_host="127.0.0.1": redis host (Redis engine)
      --redis_password="": redis auth password (Redis engine)
      --redis_port="6379": redis port (Redis engine)
      --redis_url="": redis connection URL (Redis engine)
  -w, --web="": optional path to web interface application

Global Flags:
  -h, --help=false: help for

Use " help [command]" for more information about a command.
```

To show version and exit run:

```
centrifugo version
```

But the subject of this section - configuration file. As was mentioned earlier it must be a file with valid JSON.

Let's look at configuration file example I personally use while developing Centrifugo:

```javascript
{
  "projects": [
    {
      "name": "development",
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
      ]
    }
  ],
  "log_level": "debug"
}
```

Only **projects** option is required. Projects is an array of projects. One project corresponds to your web application
that uses Centrifugo for real-time messages. For example if you are developing site `bananas.com` then you should have
something like this in configuration file:

```javascript
{
  "projects": [
    {
      "name": "bananas",
      "secret": "very-long-secret-key-for-bananas-project",
      "namespaces": []
    }
  ]
}
```

I.e project structure with one project registered. As you can see it's possible to register many projects in Centrifugo but
it's recommended to use one project for Centrifugo installation. Trust me this will make your life easier eventually.
The only exception is add the second project which is clone of first for development - so you can use production Centrifugo
instance in `bananas.com` development process.

Centrifugo also supports TOML format for configuration file:

```
centrifugo --config=config.toml
```

Where `config.toml` contains:

```
log_level = "debug"

[[projects]]

	name = "development"
	secret = "secret"

	[[projects.namespaces]]
		name = "public"
		publish = true
		watch = true
		presence = true
        join_leave = true
        history_size = 10
        history_lifetime = 30
```

I.e. the same configuration as JSON file above.

And YAML config also supported. `config.yaml`:

```
log_level: debug

projects:
  - name: development
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

In next section we will talk about project structure (projects and their namespaces) in detail. But before jumping to it
let's describe some the most important options you can configure when running Centrifugo:

* `--address` – bind your Centrifugo to specific interface address (by default `localhost`)
* `--port` – port to bind Centrifugo to (by default `8000`)
* `--engine` – engine to use - `memory` or `redis` (by default `memory`). Read more about engines later.
* `--web` – path to directory of admin web interface application to serve
* `--name` – give Centrifugo node a name - this os optional as by default Centrifugo will use hostname and port number to construct node name.

There are more command line options - we will talk about some of them later. Note that all command-line options can
be set via configuration file, but command-line options will be more valuable when set than configuration file's options.
See description of [viper](https://github.com/spf13/viper) – to see more details about configuration options priority.

As I promised above it's time to talk about project structure - projects and project options, namespaces and namespace options.