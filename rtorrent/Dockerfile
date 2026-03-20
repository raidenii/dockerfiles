FROM alpine:latest AS build

ARG TAG

RUN apk add --no-cache autoconf automake binutils build-base cppunit-dev curl-dev git libtool ncurses-dev openssl-dev zlib-dev

ENV TARGET="/target"

RUN git clone -c advice.detachedHead=false --branch ${TAG} \
    --single-branch https://github.com/rakshasa/libtorrent /src/libtorrent \
    && cd /src/libtorrent \
    && autoreconf -vfi \
    && ./configure --enable-aligned \
    && make -j$(nproc) CXXFLAGS="-w -O3 -flto -Werror=odr -Werror=lto-type-mismatch -Werror=strict-aliasing" \
    && make install \
    && make DESTDIR=${TARGET} install-strip

RUN git clone -c advice.detachedHead=false --branch ${TAG} \
    --single-branch https://github.com/rakshasa/rtorrent /src/rtorrent \
    && cd /src/rtorrent \
    && autoreconf -vfi \
    && ./configure --with-xmlrpc-tinyxml2 --with-ncurses \
    && make -j$(nproc) CXXFLAGS="-w -O3 -flto -Werror=odr -Werror=lto-type-mismatch -Werror=strict-aliasing" \
    && make DESTDIR=${TARGET} install-strip \
    && rm -rf ${TARGET}/usr/local/include ${TARGET}/usr/local/share ${TARGET}/usr/local/lib/pkgconfig ${TARGET}/usr/local/lib/*.la

FROM alpine:latest AS final

ARG TAG

ENV TARGET="/target"

RUN apk add --no-cache libncursesw libstdc++ libgcc libcurl

LABEL maintainer="r2dh" \
      org.opencontainers.image.title="rtorrent docker" \
      org.opencontainers.image.authors="r2dh" \
      org.opencontainers.image.url="https://github.com/raidenii/dockerfiles" \
      org.opencontainers.image.source="https://github.com/v2fly/v2ray-core" \
      org.opencontainers.image.description="A Docker image for rTorrent" \
      org.opencontainers.image.version=$TAG

COPY --from=build ${TARGET}/usr/local /usr/local

ENTRYPOINT ["/usr/local/bin/rtorrent"]