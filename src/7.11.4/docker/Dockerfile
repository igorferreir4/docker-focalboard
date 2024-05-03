### Github downloader
FROM alpine:latest as base

WORKDIR /ci-build
RUN apk add --no-cache --update git && \
    git clone https://github.com/mattermost/focalboard.git && \
    cd focalboard/server/services/store/sqlstore && \
    sed -i 's/Update("FileInfo")./Update("file_info")./g' ./blocks.go && \
    sed -i 's/Set("DeleteAt", model.GetMillis())./Set("delete_at", model.GetMillis())./g' ./blocks.go

### Webapp build
FROM node:22.1.0-alpine3.19 as nodebuild

RUN apk update && \
    apk add ca-certificates \
    git \
    musl-dev \
    libtool \
    zlib \
    zlib-dev \
    autoconf \
    automake \
    bash \
    g++ \
    gcc \
    musl-dev \
    libc6-compat \
    libjpeg-turbo-dev \
    libpng-dev \
    make \
    nasm && \
    cd /tmp && mkdir teste && cd teste && \
    git clone https://github.com/mattermost/focalboard && \
    cd focalboard/server/services/store/sqlstore && \
    sed -i 's/Update("FileInfo")./Update("file_info")./g' ./blocks.go && \
    sed -i 's/Set("DeleteAt", model.GetMillis())./Set("delete_at", model.GetMillis())./g' ./blocks.go && \
    cd /tmp && \
    git clone https://github.com/mattermost/focalboard && \
    cd focalboard/server/services/store/sqlstore && \
    sed -i 's/Update("FileInfo")./Update("file_info")./g' ./blocks.go && \
    sed -i 's/Set("DeleteAt", model.GetMillis())./Set("delete_at", model.GetMillis())./g' ./blocks.go && \
    cd /tmp/focalboard/webapp && \
    CPPFLAGS="-DPNG_ARM_NEON_OPT=0" npm install --no-optional && \
    npm install && npm run pack

### Go build
FROM golang:1.22.2-alpine3.19 AS gobuild

WORKDIR /go/src/focalboard
COPY --from=nodebuild /tmp/teste/focalboard/ /go/src/focalboard

# Get target architecture 
ARG TARGETOS
ARG TARGETARCH  

RUN apk update && \
    apk add ca-certificates \
    make && \
    EXCLUDE_PLUGIN=true EXCLUDE_SERVER=true EXCLUDE_ENTERPRISE=true make server-docker os=${TARGETOS} arch=${TARGETARCH}

## Final image
FROM debian:buster-slim

RUN mkdir -p /opt/focalboard/data/files
RUN chown -R nobody:nogroup /opt/focalboard

WORKDIR /opt/focalboard

COPY --from=nodebuild --chown=nobody:nogroup /tmp/focalboard/webapp/pack pack/
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/bin/docker/focalboard-server bin/
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/LICENSE.txt LICENSE.txt
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/docker/server_config.json config.json

USER nobody

EXPOSE 8000/tcp 9092/tcp

VOLUME /opt/focalboard/data

CMD ["/opt/focalboard/bin/focalboard-server"]