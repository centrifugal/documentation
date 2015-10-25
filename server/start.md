# Install Centrifugo and quick start

Go is a perfect language - it gives developers an opportunity to have single binary executable file for
application and cross-compile application on all target operating systems for distribution. This means
that all you need to get Centrifugo - [download latest release](https://github.com/centrifugal/centrifugo/releases) for you operating system, unpack it and you
are done!

Now you can see help information for Centrifugo:

```
./centrifugo -h
```

Centrifugo server node requires configuration file with secret key.
If you are new to Centrifugo then there is `genconfig` command which can generate minimal required
configuration file for you:

```bash
./centrifugo genconfig
```

It will generate secret key for you automatically and create configuration file `config.json`
in current directory (by default) so you can finally run Centrifugo instance:

```bash
./centrifugo --config=config.json
```

We will talk about configuration in detail in next sections.

You can also put or symlink `centrifugo` into your `bin` OS directory and run it from anywhere:

```bash
centrifugo --config=config.json
```