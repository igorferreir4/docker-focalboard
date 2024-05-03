### Github downloader
FROM alpine:latest as downloader
RUN apk add --no-cache wget unzip && \
    mkdir -p /tmp/focalboard && \
    cd /tmp/focalboard && \
    wget https://github.com/mattermost/focalboard/archive/refs/tags/v7.11.4.zip -O /tmp/focalboard/focalboard.zip && \
    unzip /tmp/focalboard/focalboard.zip

### Webapp build
FROM node:22.1.0 as nodebuild

WORKDIR /webapp
COPY --from=downloader /tmp/focalboard/focalboard-7.11.4/webapp/ /webapp

### 'CPPFLAGS="-DPNG_ARM_NEON_OPT=0"' Needed To Avoid Bug Described in: https://github.com/imagemin/optipng-bin/issues/118#issuecomment-1019838562
### Can be Removed when Ticket will be Closed
RUN CPPFLAGS="-DPNG_ARM_NEON_OPT=0" npm install --no-optional && \
    npm run pack

### Go build
FROM golang:1.22.2 AS gobuild

WORKDIR /go/src/focalboard
COPY --from=downloader /tmp/focalboard/focalboard-7.11.4/ /go/src/focalboard

### Get target architecture 
ARG TARGETOS
ARG TARGETARCH  

# RUN EXCLUDE_PLUGIN=true EXCLUDE_SERVER=true EXCLUDE_ENTERPRISE=true make server-docker os=${TARGETOS} arch=${TARGETARCH}
RUN EXCLUDE_PLUGIN=1 EXCLUDE_SERVER=1 EXCLUDE_ENTERPRISE=1 make server-docker os=${TARGETOS} arch=${TARGETARCH}

### Final image
FROM debian:buster-slim as final

RUN mkdir -p /opt/focalboard/data/files && \
    chown -R nobody:nogroup /opt/focalboard

WORKDIR /opt/focalboard

COPY --from=nodebuild --chown=nobody:nogroup /webapp/pack pack/
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/bin/docker/focalboard-server bin/
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/LICENSE.txt LICENSE.txt
COPY --from=gobuild --chown=nobody:nogroup /go/src/focalboard/docker/server_config.json config.json

USER nobody

EXPOSE 8000/tcp 9092/tcp

VOLUME /opt/focalboard/data

CMD ["/opt/focalboard/bin/focalboard-server"]