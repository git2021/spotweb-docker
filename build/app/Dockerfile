FROM php:8.2.6-fpm-alpine

ARG UID=1000
ARG GID=$UID

USER root

RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
    shadow \
    libjpeg-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libxml2-dev \
    libzip-dev \
    ffmpeg-dev \
    freetype-dev \
    gettext-dev \
    icu-dev \
    oniguruma-dev \
    imagemagick-dev \
    zlib-dev \
    sqlite-dev

RUN usermod --non-unique --uid $UID www-data \
    && groupmod --non-unique --gid $GID www-data

RUN apk add --no-cache --virtual .run-deps \
    supervisor \
    libjpeg-turbo \
    libpng \
    libwebp \
    libxml2 \
    libzip \
    ffmpeg \
    freetype \
    icu \
    oniguruma \
    imagemagick \
    zlib \
    sqlite \
    libcap \
    mariadb-client \
    nodejs \
    npm \
    ffmpeg \
    jpegoptim \
    optipng \
    pngquant \
    gifsicle

RUN cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini" \
    && sed -i "s/error_reporting = .*/error_reporting = E_ALL/" "$PHP_INI_DIR/php.ini" \
    && sed -i "s/display_errors = .*/display_errors = On/" "$PHP_INI_DIR/php.ini" \
    && sed -i "s/memory_limit = .*/memory_limit = 512M/" "$PHP_INI_DIR/php.ini" \
    && sed -i "s/upload_max_filesize = .*/upload_max_filesize = 100M/" "$PHP_INI_DIR/php.ini" \
    && sed -i "s/post_max_size = .*/post_max_size = 100M/" "$PHP_INI_DIR/php.ini" \
    && sed -i "s/;date.timezone.*/date.timezone = UTC/" "$PHP_INI_DIR/php.ini" \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg --with-webp \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install -j$(nproc) bcmath \
    && docker-php-ext-install -j$(nproc) exif \
    && docker-php-ext-install -j$(nproc) fileinfo \
    && docker-php-ext-install -j$(nproc) gettext \
    && docker-php-ext-install -j$(nproc) intl \
    && docker-php-ext-install -j$(nproc) mbstring \
    && docker-php-ext-install -j$(nproc) pcntl \
    && docker-php-ext-install -j$(nproc) pdo_mysql \
    && docker-php-ext-install -j$(nproc) pdo_sqlite \
    && docker-php-ext-install -j$(nproc) simplexml \
    && docker-php-ext-install -j$(nproc) zip \
    && pecl install apcu \
    && pecl install imagick \
    && docker-php-ext-enable apcu \
    && docker-php-ext-enable imagick \
    && docker-php-ext-enable opcache \
    && apk del .build-deps

RUN setcap "cap_net_bind_service=+ep" $(which php) \
    && setcap "cap_net_bind_service=+ep" $(which php-fpm)

COPY runtimes/supervisord.conf /etc/supervisord.conf
COPY runtimes/www.conf /usr/local/etc/php-fpm.d/www.conf

WORKDIR /src/spotweb

STOPSIGNAL SIGQUIT

EXPOSE 9000

CMD ["supervisord"]
