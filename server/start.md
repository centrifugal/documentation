# Install Centrifugo and quick start

Go is a perfect language - it gives developers an opportunity to have single binary executable file for
application and cross-compile application on all target operating systems for distribution. This means
that all you need to get Centrifugo - [download latest release](https://github.com/centrifugal/centrifugo/releases) for you operating system, unpack it and you
are done!

Now you can see help information for centrifugo:

```
./centrifugo -h
```

Centrifugo node requires configuration file with your project name and its secret key. If you
are new to Centrifugo then there is `genconfig` command which can generate minimal required
configuration file for you:

```bash
./centrifugo genconfig
```

It will ask you to enter your web project name, generate secret key for it automatically
and create configuration file `config.json` in current directory (by default) so you can
finally run Centrifugo instance:

```bash
./centrifugo --config=config.json
```

We will talk about configuration in detail in next section.

You can also put or symlink `centrifugo` into your `bin` OS directory and run it
from anywhere:

```bash
centrifugo --config=config.json
```