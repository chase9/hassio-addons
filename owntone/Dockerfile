ARG BUILD_FROM=hassioaddons/base:8.0.0
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

#adapted from sretalla's/kevineye's dockerfiles
# hadolint ignore=DL3003
RUN apk update 
RUN apk add --no-cache \
 libcrypto1.1 \
 libssl1.1 
RUN apk add --no-cache \
 musl \
 musl-dev
 
RUN apk add --no-cache --virtual deps1 \
        alsa-lib-dev \
        autoconf \
        automake \
        cmake \
        avahi-dev \
        bash \
        bsd-compat-headers \
        build-base \
        confuse-dev \
        curl \
        curl-dev \
        ffmpeg-dev \
        file \
        git \
        gnutls-dev \
        gperf \
        json-c-dev \
        libevent-dev \
        libgcrypt-dev \
        libplist-dev \
        libsodium-dev \
        libtool \
        libunistring-dev \
        openjdk7-jre-base \
        protobuf-c-dev \
        sqlite-dev
        
RUN apk add --no-cache --virtual=deps2 --repository http://nl.alpinelinux.org/alpine/edge/testing --repository http://nl.alpinelinux.org/alpine/edge/community \
        libantlr3c-dev \
        mxml \
        mxml-dev 
        
RUN apk add --no-cache \
        avahi \
        confuse \
        dbus \
        ffmpeg \
        json-c \
        libcurl \
        libevent \
        libgcrypt \
        libplist \
        libsodium \
        libunistring \
        libuv \
        libuv-dev \
        protobuf-c \
        sqlite \
        sqlite-libs \
        openssl 
        
RUN apk add --no-cache --repository http://nl.alpinelinux.org/alpine/edge/testing \
        libantlr3c \
        mxml 
 
RUN curl -L -o /tmp/antlr-3.4-complete.jar http://www.antlr3.org/download/antlr-3.4-complete.jar && \
    echo '#!/bin/bash' > /usr/local/bin/antlr3 && \
    echo 'exec java -cp /tmp/antlr-3.4-complete.jar org.antlr.Tool "$@"' >> /usr/local/bin/antlr3 && \
    chmod 775 /usr/local/bin/antlr3 

RUN cd /tmp && \
    git clone --branch "v3.2.2" --depth=1 https://github.com/warmcat/libwebsockets.git && \
    cd /tmp/libwebsockets && \
    cmake ./ \
        -DCMAKE_BUILD_TYPE=MinSizeRel \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_VERBOSE_MAKEFILE=TRUE \
        -DLWS_IPV6=ON \
        -DLWS_STATIC_PIC=ON \
        -DLWS_UNIX_SOCK=OFF \
        -DLWS_WITH_LIBUV=ON \
        -DLWS_WITH_SHARED=ON \
        -DLWS_WITHOUT_TESTAPPS=ON && \
    make && \
    make install 

RUN cd /tmp && \
    git clone https://github.com/owntone/owntone-server.git && \
    cd /tmp/owntone-server && \
    autoreconf -fi && \
    ./configure \
      --enable-itunes \
      --with-libwebsockets && \
    make && \
    make install 

RUN apk del --purge deps1 deps2 && \
    rm -rf /usr/local/bin/antlr3 /tmp/* 

RUN cd /usr/local/etc \
 && sed -i -e 's/\(uid.*=.*\)/uid = "root"/g' owntone.conf \
 && sed -i s#"ipv6 = yes"#"ipv6 = no"#g owntone.conf \
 && sed -i s#/srv/music#/config/owntone/music#g owntone.conf \
 && sed -i s#/usr/local/var/cache/owntone/songs3.db#/config/owntone/cache/songs3.db#g owntone.conf \
 && sed -i s#/usr/local/var/cache/owntone/cache.db#/config/owntone/cache/cache.db#g owntone.conf \
 && sed -i s#/usr/local/var/log/owntone.log#/dev/stdout#g owntone.conf \
 && sed -i "/websocket_port\ =/ s/# *//" owntone.conf \
 && sed -i "/trusted_networks\ =/ s/# *//" owntone.conf \
 && sed -i "/pipe_autostart\ =/ s/# *//" owntone.conf \
 && sed -i "/db_path\ =/ s/# *//" owntone.conf \
 && sed -i "/cache_path\ =/ s/# *//" owntone.conf

VOLUME /etc/localtime /etc/localtime:ro

# Copy root filesystem
COPY rootfs /

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="Owntone Server" \
    io.hass.description="The owntone server program" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="John Dowling <john.patrick.dowling@gmail.com>" \
    org.label-schema.description="The owntone server program" \
    org.label-schema.build-date=${BUILD_DATE} \
    org.label-schema.name="owntone Server" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.url="" \
    org.label-schema.usage="https://github.com/johnpdowling/hassio-addons/tree/master/owntone/README.md" \
    org.label-schema.vcs-ref=${BUILD_REF} \
    org.label-schema.vcs-url="https://github.com/johnpdowling/hassio-addons/owntone" \
    org.label-schema.vendor="JPD Hass.io Add-ons"
