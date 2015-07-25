# Configuration overview

Centrifugo expects JSON, TOML or YAML formats as format of configuration file.
Thanks to brilliant Go library for application configuration - [viper](https://github.com/spf13/viper).

But first let's inspect all available command-line options:

```bash
centrifugo -h
```

You should see something like this as output:

```
Centrifuge + GO = Centrifugo – harder, better, faster, stronger

Usage:
   [flags]
   [command]

Available Commands:
  version     Centrifugo version number
  checkconfig Check configuration file
  genconfig   Generate simple configuration file to start with
  help        Help about any command

Flags:
  -a, --address="": address to listen on
  -c, --config="config.json": path to config file
  -d, --debug=false: debug mode - please, do not use it in production
  -e, --engine="memory": engine to use: memory or redis
      --insecure=false: start in insecure mode
      --log_file="": optional log file - if not specified all logs go to STDOUT
      --log_level="info": set the log level: debug, info, error, critical, fatal or none
  -n, --name="": unique node name
  -p, --port="8000": port to bind to
      --redis_api=false: enable Redis API listener (Redis engine)
      --redis_db="0": redis database (Redis engine)
      --redis_host="127.0.0.1": redis host (Redis engine)
      --redis_password="": redis auth password (Redis engine)
      --redis_pool=256: Redis pool size (Redis engine)
      --redis_port="6379": redis port (Redis engine)
      --redis_url="": redis connection URL (Redis engine)
      --ssl=false: accept SSL connections. This requires an X509 certificate and a key file
      --ssl_cert="": path to an X509 certificate file
      --ssl_key="": path to an X509 certificate key
  -w, --web="": optional path to web interface application

Global Flags:
  -h, --help=false: help for

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
  "project_name": "development",
  "project_secret": "secret",
  "project_namespaces": [
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

Only **project_name** and **project_secret** options are required.

Project corresponds to your web application that uses Centrifugo for real-time messages. You must give it a name
and set secret key.

You will know about other project options such as `namespaces` in next sections.

So the **minimal configuration file required** is:

```javascript
{
  "project_name": "development",
  "project_secret": "secret"
}
```

### TOML

Centrifugo also supports TOML format for configuration file:

```
centrifugo --config=config.toml
```

Where `config.toml` contains:

```
log_level = "debug"

project_name = "development"
project_secret = "secret"

[[project_namespaces]]
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

project_name: development
project_secret: secret
project_namespaces:
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

There is also a way to register multiple projects in Centrifugo.

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

**projects** in this case is an array of projects. One project corresponds to your web application that
uses Centrifugo for real-time messages. **Although it's possible to register many projects in Centrifugo
it's recommended to use only one project for Centrifugo installation.** Trust me this will make your life
easier eventually. **The only exception** is add the second project which is clone of first for development –
so you can use production Centrifugo instance in development process.

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

It will generate the simplest configuration file for you – will ask you to enter your web project name and
generate secret key for it automatically.

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

### project settings reload

Centrifugo can reload project settings and some other configuration options on the fly.

To reload you must send `HUP` signal to centrifugo process:

```bash
kill -HUP <CENTRIFUGO_PROCESS_PID>
```
