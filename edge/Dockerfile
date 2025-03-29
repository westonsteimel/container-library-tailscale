ARG VERSION="main"
ARG REVISION="bf8c8e9e8989d6d7e4b678e6647073dd39069ac4"

FROM golang:alpine AS builder

ARG VERSION
ARG REVISION

ENV CGO_ENABLED=0
ENV GOOS=linux
ENV GO111MODULE=on

RUN apk upgrade && apk --no-cache add \
	ca-certificates \
    git \
    make

RUN mkdir -p /go/src/tailscale \
    && git clone --branch "${VERSION}" https://github.com/tailscale/tailscale.git /go/src/tailscale \
	&& cd /go/src/tailscale \
    && git reset --hard "${REVISION}" \
    && go install -tags=xversion \
    -ldflags "\
    -X tailscale.com/version.Long=${VERSION} \
    -X tailscale.com/version.Short=${VERSION} \
    -X tailscale.com/version.GitCommit=${REVISION} \
    -s -w" \
    -a ./cmd/tailscale* \
	&& cp -vr /go/bin/* /usr/local/bin/ \
	&& if [$TARGETPLATFORM = "linux/amd64"]; then apk add --no-cache upx && upx --lzma --best /usr/local/bin/tailscale*; fi 

FROM alpine:latest

ARG VERSION
ARG REVISION

ENV TAILSCALE_VERSION="${VERSION}" \
    TAILSCALE_REVISION="${REVISION}"

RUN apk add --no-cache \
    ca-certificates \
    iptables \
    iproute2

COPY --from=builder /usr/local/bin/* /usr/local/bin/

LABEL org.opencontainers.image.title="tailscale" \
    org.opencontainers.image.description="tailscale in Docker" \ 
    org.opencontainers.image.revision="${REVISION}" \
    org.opencontainers.image.version="${VERSION}"

