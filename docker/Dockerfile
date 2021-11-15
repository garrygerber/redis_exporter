ARG GOARCH
#
# build container
#
FROM --platform=linux/amd64 golang:1.17-alpine as builder
WORKDIR /go/src/github.com/oliver006/redis_exporter/

ADD . /go/src/github.com/oliver006/redis_exporter/

ARG SHA1="[no-sha]"
ARG TAG="[no-tag]"

# RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
#   && case "${dpkgArch##*-}" in \
#     amd64) ARCH='x64';; \
#     ppc64el) ARCH='ppc64le';; \
#     s390x) ARCH='s390x';; \
#     arm64) ARCH='arm64';; \
#     armhf) ARCH='armv7l';; \
#     i386) ARCH='x86';; \
#     *) echo "unsupported architecture"; exit 1 ;; \
#   esac
# RUN echo $ARCH
RUN apk --no-cache add ca-certificates
RUN BUILD_DATE=$(date +%F-%T) && CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -o /redis_exporter \
    -ldflags  "-s -w -extldflags \"-static\" -X main.BuildVersion=$TAG -X main.BuildCommitSha=$SHA1 -X main.BuildDate=$BUILD_DATE" .

RUN [ $GOARCH = "amd64" ]  && /redis_exporter -version || ls -la /redis_exporter

#
# scratch release container
#
FROM --platform=linux/arm64 scratch as scratch

COPY --from=builder /redis_exporter /redis_exporter
COPY --from=builder /etc/ssl/certs /etc/ssl/certs
COPY --from=builder /etc/nsswitch.conf /etc/nsswitch.conf

# Run as non-root user for secure environments
USER 59000:59000

EXPOSE     9121
ENTRYPOINT [ "/redis_exporter" ]


#
# Alpine release container
#
FROM --platform=linux/arm64 alpine as alpine

COPY --from=builder /redis_exporter /redis_exporter
COPY --from=builder /etc/ssl/certs /etc/ssl/certs

# Run as non-root user for secure environments
USER 59000:59000

EXPOSE     9121
ENTRYPOINT [ "/redis_exporter" ]
