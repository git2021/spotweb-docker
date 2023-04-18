FROM nginxinc/nginx-unprivileged:mainline-alpine

ARG UID=1000
ARG GID=$UID

USER root

RUN apk add --no-cache --virtual .build-deps \
    shadow \
    git \
    gcc \
    libc-dev \
    make \
    openssl-dev \
    pcre2-dev \
    zlib-dev \
    linux-headers \
    libxslt-dev \
    perl-dev \
    bash \
    alpine-sdk \
    findutils \
    brotli-dev \
    ffmpeg-dev

RUN usermod --non-unique --uid $UID nginx \
    && groupmod --non-unique --gid $GID nginx

RUN apk add --no-cache --virtual .run-deps \
    brotli \
    ffmpeg \
    fdk-aac

RUN mkdir -p /usr/local/src && cd /usr/local/src \
    && wget "https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz" -O nginx.tar.gz \
    && set -x \
    && tar xzvf nginx.tar.gz \
    && rm -rf nginx.tar.gz \
    && mv /usr/local/src/nginx-${NGINX_VERSION} /usr/local/src/nginx

RUN cd /usr/local/src \
    && git clone --depth 1 https://github.com/google/ngx_brotli.git \
    && MODULEDIR=$(pwd)/ngx_brotli \
    && cd /usr/local/src/nginx \
    && ./configure --with-compat $CONFARGS --add-dynamic-module=$MODULEDIR \
    && make \
    && make install

RUN apk del .build-deps

COPY runtimes/. /etc/nginx

EXPOSE 8080
EXPOSE 4443

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]
