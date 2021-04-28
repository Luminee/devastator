ARG PHP_VERSION
ARG ALPINE_VERSION

FROM hyperf/hyperf:${PHP_VERSION}-alpine-v${ALPINE_VERSION}-base

ARG SWOOLE_VERSION
ENV PHPIZE_DEPS="autoconf dpkg-dev dpkg file g++ gcc libc-dev make php7-dev php7-pear pkgconf re2c pcre-dev pcre2-dev zlib-dev libtool automake"

RUN set -ex \
    && apk update \
    # for swoole extension libaio linux-headers
    && apk add --no-cache libstdc++ openssl git bash \
    && apk add --no-cache --virtual .build-deps ${PHPIZE_DEPS} libaio-dev openssl-dev curl-dev \
    # download
    && cd /tmp \
    && curl -SL "https://github.com/swoole/swoole-src/archive/${SWOOLE_VERSION}.tar.gz" -o swoole.tar.gz \
    && ls -alh \
    # php extension:swoole
    && cd /tmp \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && ln -s /usr/bin/phpize7 /usr/local/bin/phpize \
    && ln -s /usr/bin/php-config7 /usr/local/bin/php-config \
    && ( \
        cd swoole && phpize \
        && ./configure --enable-openssl --enable-http2 --enable-swoole-curl --enable-swoole-json \
        && make -s -j$(nproc) && make install \
    ) \
    && echo "memory_limit=1G" > /etc/php7/conf.d/00_default.ini \
    && echo "opcache.enable_cli = 'On'" >> /etc/php7/conf.d/00_opcache.ini \
    && echo "extension=swoole.so" > /etc/php7/conf.d/50_swoole.ini \
    && echo "swoole.use_shortname = 'Off'" >> /etc/php7/conf.d/50_swoole.ini \
    # install composer
    && wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/2.0.2/composer.phar \
    && chmod u+x /usr/local/bin/composer \
    # ---------- clear works ----------
    && apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/local/bin/php* \
    && echo -e "\033[42;37m Swoole Build Completed :).\033[0m\n"