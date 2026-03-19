FROM golang:alpine AS build

WORKDIR /tmp

ARG TAG

RUN apk add --no-cache ca-certificates git upx

RUN git clone -c advice.detachedHead=false --branch ${TAG} \
    --single-branch https://repo.or.cz/dnstt.git src/dnstt \
    && cd src/dnstt \
    && go mod download \
    && CGO_ENABLED=0 GOOS=linux go build -o /tmp/bin/dnstt-server \
       -trimpath -ldflags "-s -w -buildid=" ./dnstt-server \
    && upx --ultra-brute --lzma /tmp/bin/dnstt-server

RUN mkdir -p ./etc \
    && echo "dnstt:x:7000:7000::/nonexistent:/sbin/nologin" >> ./etc/passwd \
    && echo "dnstt:!:::::::" >> ./etc/shadow \
    && echo "dnstt:x:7000:" >> ./etc/group \
    && echo "dnstt:!::" >> ./etc/groupshadow \
    && chmod 0400 ./etc/shadow ./etc/groupshadow

FROM scratch AS final

ARG TAG

LABEL maintainer="r2dh" \
      org.opencontainers.image.title="dnstt Docker" \
      org.opencontainers.image.authors="r2dh" \
      org.opencontainers.image.url="https://github.com/raidenii/dnstt-docker" \
      org.opencontainers.image.source="https://repo.or.cz/dnstt.git" \
      org.opencontainers.image.description="A Docker image for dnstt" \
      org.opencontainers.image.version=$TAG

COPY --from=build /tmp/etc/* /etc/
COPY --from=build --chown=dnstt --chmod=755 /tmp/bin/dnstt-server /bin/dnstt-server

USER dnstt

ENTRYPOINT ["/bin/dnstt-server"]
