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

### Alternative alpine-based Dockerfile

Centrifugo docker image based on centos 7 image and it's rather big in size (about 250mb).

[Egor Yurtaev](https://github.com/yurtaev) contributed Dockerfile based on alpine base image which results in much smaller Centrifugo image (about 25mb):

```
FROM gliderlabs/alpine:3.2

ENV VERSION 1.1.0

ENV DOWNLOAD https://github.com/centrifugal/centrifugo/releases/download/v$VERSION/centrifugo-$VERSION-linux-amd64.zip

RUN apk --update add unzip \
    && addgroup -S centrifugo && adduser -S -G centrifugo centrifugo \
    && mkdir /centrifugo && chown centrifugo:centrifugo /centrifugo \
    && mkdir /var/log/centrifugo && chown centrifugo:centrifugo /var/log/centrifugo

ADD ${DOWNLOAD} /tmp/centrifugo.zip

RUN unzip -jo /tmp/centrifugo.zip -d /tmp/ \
    && mv /tmp/centrifugo /usr/bin/centrifugo \
    && rm -f /tmp/centrifugo.zip \
    && apk del unzip \
    && rm -rf /var/cache/apk/*

VOLUME ["/centrifugo", "/var/log/centrifugo"]

WORKDIR /centrifugo

USER centrifugo

CMD ["centrifugo"]

EXPOSE 8000
```

You can run container in the same way:

```
docker run --ulimit nofile=65536:65536 -v /your/host/dir/containing/config/file:/centrifugo -p 8000:8000 CONTAINER_NAME centrifugo -c config.json
```

Note that docker allows to set nofile limits in command-line arguments.