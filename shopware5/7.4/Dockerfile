FROM php:7.4-fpm-alpine3.13
LABEL author='Linus Lotz<l.lotz@reply.de>'
LABEL maintainer='Manuel Mayer<m.mayer@reply.de>'
ENV RUN_DEPS="bash freetype git icu-libs libgd libintl libjpeg-turbo libpng libwebp libxml2 libxpm libzip mariadb-connector-c openssh-client patch postgresql-libs su-exec zlib" \
    BUILD_DEPS="${PHPIZE_DEPS} freetype-dev icu-dev libjpeg-turbo-dev libpng-dev libwebp-dev libxml2-dev libxpm-dev libzip-dev mariadb-connector-c-dev postgresql-dev zlib-dev" \
    PECL_EXTS="apcu redis" \
    PHP_EXTS="gd intl mysqli pcntl pdo_mysql pdo_pgsql soap zip" \
    ALPINE_VERSION="3.13"
ENV PHP_JPEG_DIR="/usr" \
    PHP_PNG_DIR="/usr" \
    PHP_ZLIB_DIR="/usr" \
    PHP_WEBP_DIR="/usr" \
    PHP_INI_SCAN_DIR="/usr/local/etc/php/conf.d/:/usr/local/etc/php/conf-override.d/" \
    # fixes iconv
    LD_PRELOAD="/usr/lib/preloadable_libiconv.so"

RUN apk add --no-cache --repository https://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VERSION}/community gnu-libiconv

RUN    apk update \
    && apk upgrade \
    && apk add --no-cache $RUN_DEPS \
    && apk add -t .build-deps $BUILD_DEPS \
    && if [ -z "$(echo "$PHP_EXTS" |sed '/^gd\| gd/d')" ]; then \
            if [ "${PHP_VERSION%.*}" == "7.4" -o "${PHP_VERSION%.*}" == "8.0" -o "${PHP_VERSION%.*}" == "8.1" ]; then \
                docker-php-ext-configure gd --enable-gd \
                --with-webp \
                --with-jpeg \
                --with-xpm \
                --with-freetype \
                --enable-gd-jis-conv \
            ; fi && \
            if [ "${PHP_VERSION%.*}" == "7.3" -o "${PHP_VERSION%.*}" == "7.2" ]; then \
                docker-php-ext-configure gd \
                --with-freetype-dir=/usr \
                --with-jpeg-dir=/usr \
                --with-webp-dir=/usr \
                --with-zlib-dir=/usr \
                --with-png-dir=/usr \
                --with-xpm-dir=/usr \
                --enable-gd-jis-conv \
            ; fi \
        ; fi \
    && docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) $PHP_EXTS \
    && docker-php-ext-enable --ini-name 05-opcache.ini opcache \
    && pecl channel-update pecl.php.net \
    && pecl install -o -f $PECL_EXTS \
    && docker-php-ext-enable $PECL_EXTS \
    && cp "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini" \
    && pecl clear-cache \
    && apk del .build-deps \
    # Clear
    && rm -rf /tmp/* /var/cache/apk/*

COPY --from=composer:1 /usr/bin/composer /usr/bin/composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer2
