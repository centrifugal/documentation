# Docker image

Centrifugo server has docker image [available on Docker Hub](https://registry.hub.docker.com/u/fzambia/centrifugo/).

```
docker pull fzambia/centrifugo
```

Run:

```bash
docker run -v /your/host/dir/containing/config/file:/centrifugo -p 8000:8000 fzambia/centrifugo centrifugo -c config.json
```

To run with admin web interface:

```bash
docker run -v /your/host/dir/containing/config/file:/centrifugo -p 8000:8000 fzambia/centrifugo centrifugo -c config.json -w
```
