FROM php:7.1-alpine

MAINTAINER Simon Forsman <simon@urb-it.com>

# Add configs
ADD ./lumen.ini /usr/local/etc/php/conf.d
ADD ./worker.conf /etc/supervisor/worker.conf

# Environment variables
ENV ALPINE_VERSION 3.4
ENV PHP_API_VERSION 20160303
ENV NEWRELIC_VERSION 6.9.0.182
ENV PYTHON_VERSION=2.7.12-r0
ENV PY_PIP_VERSION=8.1.2-r0
ENV SUPERVISOR_VERSION=3.3.1
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
    icu-dev \
    libpng \
    libpng-dev \
    libjpeg-turbo \
    libjpeg-turbo-dev \
    imagemagick-dev \
    imagemagick \
    ca-certificates \
&& apk --no-cache --update --repository http://dl-cdn.alpinelinux.org/alpine/edge/main/ add \
    libressl \
    libressl-dev \
&& apk --no-cache --update --repository http://dl-3.alpinelinux.org/alpine/edge/community/ add \
    php7-gd \
    php7-sockets \
    php7-zlib \
    php7-intl \
    php7-opcache \
    php7-bcmath \
    python=$PYTHON_VERSION \
    py-pip=$PY_PIP_VERSION \
&& apk --no-cache --update --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing/ --repository http://dl-cdn.alpinelinux.org/alpine/v$ALPINE_VERSION/main/ add \
    rabbitmq-c \
    rabbitmq-c-dev \
&& docker-php-ext-configure intl \
&& docker-php-ext-configure gd \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
&& pecl install \
    imagick-$IMAGICK_VERSION \
    amqp-$AMQP_VERSION \
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
    amqp \
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
    libressl-dev \
    rabbitmq-c-dev \
    libtool \
&& rm -rf /var/cache/apk/* /tmp/* ./newrelic-php5-$NEWRELIC_VERSION-linux-musl*

# Install supervisor
RUN pip install supervisor==$SUPERVISOR_VERSION

# Workaround for piping PHP-FPM worker logs to the container log output
# https://bugs.php.net/bug.php?id=71880
RUN mkfifo /tmp/stdout && chmod 777 /tmp/stdout

WORKDIR /var/www/application

CMD sh -c "supervisord --nodaemon --configuration /etc/supervisor/worker.conf"
