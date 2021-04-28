ARG PHP_VERSION
FROM php:${PHP_VERSION}-fpm

RUN echo "deb http://mirrors.aliyun.com/debian/ buster main non-free contrib" > /etc/apt/sources.list

RUN apt-get update && apt-get install -y --fix-missing apt-utils gnupg
RUN apt-get install -y libpq-dev libxslt1-dev
RUN docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql
RUN docker-php-ext-configure soap --enable-soap && \
    docker-php-ext-configure bcmath --enable-bcmath
RUN apt-get update && apt-get install -y zlib1g-dev libzip-dev 
RUN rm -rf /var/lib/apt/lists/* && docker-php-ext-install zip
RUN apt-get update \
    && apt-get install -y libwebp-dev libjpeg-dev libpng-dev libfreetype6-dev
RUN docker-php-source extract && cd /usr/src/php/ext/gd
RUN docker-php-ext-configure gd --with-webp-dir=/usr/include/webp \ 
    --with-jpeg-dir=/usr/include --with-png-dir=/usr/include \
    --with-freetype-dir=/usr/include/freetype2 
RUN docker-php-ext-install pdo pdo_mysql pdo_pgsql pgsql intl soap bcmath xsl sockets gd  

# composer

RUN php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer \
    && composer config -g repo.packagist composer https://packagist.phpcomposer.com

# swoole
COPY ./extensions /tmp/extensions
RUN cd /tmp/extensions && mkdir swoole \
    && tar -xf swoole-src-1.9.15.tar.gz -C swoole --strip-components=1 \
    && ( cd swoole \ 
    && phpize && ./configure && make $mc && make install ) \
    && docker-php-ext-enable swoole

RUN cd /tmp/extensions && mkdir hprose \
    && tar -xf hprose-pecl-1.8.0.tar.gz -C hprose --strip-components=1 \
    && (cd hprose && phpize && ./configure --enable-hprose=shared && make $mc && make install) \
    && docker-php-ext-enable hprose
