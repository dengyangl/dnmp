FROM php:7.2-fpm

ENV PHPREDIS_VERSION 3.1.3

#RUN sed -i "s/archive.ubuntu./mirrors.aliyun./g" /etc/apt/sources.list
#RUN sed -i "s/deb.debian.org/mirrors.aliyun.com/g" /etc/apt/sources.list
#RUN sed -i "s/security.debian.org/mirrors.aliyun.com\/debian-security/g" /etc/apt/sources.list

# install redis and mysql
RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
    && tar xfz /tmp/redis.tar.gz \
    && rm -r /tmp/redis.tar.gz \
    && mkdir -p /usr/src/php/ext \
    && mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
        pdo pdo_mysql \
        bcmath

# install php extension: iconv,gd,zip,grpc,sockets,swoole
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
        libmemcached-dev zlib1g-dev \
        automake \
        cmake \
        g++ \
        git \
        libicu-dev \
        libmagickwand-dev \
        librabbitmq-dev \
        libreadline-dev \
        pkg-config \
        ssh-client \
        supervisor \
        zip \
        unzip \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && pecl install grpc \
    && pecl install protobuf \
    && rm -rf /usr/src/php \
    && docker-php-ext-install \
         sockets \
         intl \
    && pecl install swoole

# install composer
RUN curl -sS https://getcomposer.org/installer \
    | php -- --install-dir=/usr/local/bin --filename=composer