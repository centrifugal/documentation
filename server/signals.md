# Signal handling

This is a chapter about sending operating system signals to Centrifugo.

### Channel options and configuration reload.

Centrifugo can reload channel options and some other configuration options on the fly.

To reload you must send `HUP` signal to centrifugo process:

```bash
kill -HUP <CENTRIFUGO_PROCESS_PID>
```

Some options can not be reloaded because they are used on Centrifugo start. For example various engine
options, interface and port to listen etc.