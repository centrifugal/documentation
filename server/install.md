# How to install Centrifugo

Go is a perfect language - it gives developers an opportunity to have single binary executable file for
application and cross-compile application on all target operating systems for distribution. This means
that all you need to get Centrifugo - [download latest release](https://github.com/centrifugal/centrifugo/releases) for you operating system, unpack it and you
are done!

To run Centrifugo instance just write:

```bash
./centrifugo --config=config.json
```

Configuration file required to launch, we will talk about it in next section.

You can also put or symlink `centrifugo` into your `bin` OS directory and run it
from anywhere:

```bash
centrifugo --config=config.json
```