FROM php:7.1-fpm-alpine

MAINTAINER Sebastian Mandrean <sebastian@urb-it.com>

# PHP
ADD ./logging.ini /usr/local/etc/php/conf.d
ADD ./lumen.ini /usr/local/etc/php/conf.d
ADD ./newrelic.ini /usr/local/etc/php/conf.d

# PHP-FPM
ADD ./lumen.pool.conf /usr/local/etc/php-fpm.d
RUN rm /usr/local/etc/php-fpm.d/www.conf*

# Environment variables
ENV ALPINE_VERSION 3.4
ENV PHP_API_VERSION 20160303
ENV NEWRELIC_VERSION 6.9.0.182
ENV IMAGICK_VERSION=3.4.3
ENV AMQP_VERSION=1.7.1
ENV MONGODB_VERSION=1.2.5

# Download & Install New Relic
RUN curl -#SL https://download.newrelic.com/php_agent/archive/$NEWRELIC_VERSION/newrelic-php5-$NEWRELIC_VERSION-linux-musl.tar.gz | tar xzf - \
&& mv ./newrelic-php5-$NEWRELIC_VERSION-linux-musl/agent/x64/newrelic-$PHP_API_VERSION.so /usr/local/lib/php/extensions/no-debug-non-zts-$PHP_API_VERSION/newrelic.so \
&& mv ./newrelic-php5-$NEWRELIC_VERSION-linux-musl/daemon/newrelic-daemon.x64 /usr/bin/newrelic-daemon \
&& chmod 775 /usr/bin/newrelic-daemon \
&& mkdir /var/log/newrelic

# Install & clean up dependencies
RUN apk --no-cache --update --repository http://dl-cdn.alpinelinux.org/alpine/v$ALPINE_VERSION/main/ add \
    autoconf \
    curl \
    build-base \
    libtool \
    icu \
    icu-libs \
    icu-dev \
    libpng \
    libpng-dev \
    libjpeg-turbo \
    libjpeg-turbo-dev \
    imagemagick-dev \
    imagemagick \
    ca-certificates \
    openssl \
    openssl-dev \
&& apk --no-cache --update --repository http://dl-3.alpinelinux.org/alpine/v3.5/community/ add \
    php7-gd \
    php7-sockets \
    php7-zlib \
    php7-intl \
    php7-opcache \
    php7-bcmath \
&& docker-php-ext-configure intl \
&& docker-php-ext-configure gd \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
&& pecl install \
    imagick-$IMAGICK_VERSION \
    mongodb-$MONGODB_VERSION \
&& docker-php-ext-install \
    pdo_mysql \
    sockets \
    gd \
    intl \
    opcache \
    bcmath \
&& docker-php-ext-enable \
    imagick \
    mongodb \
&& apk --no-cache del \
    wget \
    icu-dev \
    libpng-dev \
    libjpeg-turbo-dev \
    imagemagick-dev \
    tar \
    autoconf \
    build-base \
    libtool \
&& rm -rf /var/cache/apk/* /tmp/* ./newrelic-php5-$NEWRELIC_VERSION-linux-musl*

CMD sh -c "newrelic-daemon && php-fpm | tail -f /tmp/stdout"

WORKDIR /var/www/application

EXPOSE 9000
