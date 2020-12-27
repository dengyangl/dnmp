FROM php:7.2-fpm

#ENV PHPREDIS_VERSION 3.1.3
ENV PHPREDIS_VERSION 4.0.1
#ENV SWOOLE_VERSION 4.3.3
ENV SWOOLE_VERSION 4.5.2

ENV MOSQUITTO_VERSION 1.4.11

ENV PHP_RDKAFKA_VERSION 3.0.1

#ENV librdkafka php-rdkafka
#ENV LIBRDKAFKA 0.9.4
#ENV PROTOC_VERSION 3.9.1


RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN apt-get clean

# install redis and mysql
RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
    && tar xfz /tmp/redis.tar.gz \
    && rm -r /tmp/redis.tar.gz \
    && mkdir -p /usr/src/php/ext \
    && mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis \
    && docker-php-ext-install redis \
        pdo pdo_mysql \
        bcmath

# install php extension: iconv,gd,zip,grpc,sockets,swoole,yaconf,mosquitto(mqtt),pcntl...
RUN apt-get update && apt-get install -y \
        vim \
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
        unzip \
        inetutils-ping \
        curl \
        wget \
        telnet \
        libz-dev \
        libssl-dev \
        libnghttp2-dev \
        libpcre3-dev \
        #libhiredis-devel \
        # mqtt extension
        libc-ares-dev \
        # mqtt extension
        uuid-dev \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
#    && wget https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOC_VERSION}/protoc-${PROTOC_VERSION}-linux-x86_64.zip \
#           unzip protoc-${PROTOC_VERSION}-linux-x86_64.zip \
#           # 该目录下会增加包和文件夹 bin包 include包 readme.txt
#           #ls
#           # 将bin包下的protoc复制到/usr/local/bin下
#           cp /bin/protoc /usr/local/bin \
    && pecl install grpc \
    # 接口描述语言,相关处理工具主要是protoc
    && pecl install protobuf \
    && rm -rf /usr/src/php \
    && docker-php-ext-install \
         sockets \
         intl \
         zip \
         # pcntl_fork
         pcntl \
    && pecl install yaconf \
    # mqtt extension
    && curl -s https://mosquitto.org/files/source/mosquitto-$MOSQUITTO_VERSION.tar.gz > mosquitto-$MOSQUITTO_VERSION.tgz \
        && tar -zxvf mosquitto-$MOSQUITTO_VERSION.tgz \
        ###删除压缩包
        && rm -r mosquitto-$MOSQUITTO_VERSION.tgz \
        && cd mosquitto-$MOSQUITTO_VERSION && make && make install \
        && pecl install Mosquitto-alpha \
        && echo "extension=mosquitto.so" > /usr/local/etc/php/conf.d/docker-php-ext-mosquitto.ini \
        && ln -s /usr/local/lib/libmosquitto.so.1 /usr/lib/libmosquitto.so.1 && ldconfig \
        && cp /etc/mosquitto/mosquitto.conf.example /etc/mosquitto/mosquitto.conf && echo "user root" >> /etc/mosquitto/mosquitto.conf


###可用

# Swoole extension
#RUN wget https://github.com/swoole/swoole-src/archive/v${SWOOLE_VERSION}.tar.gz -O swoole.tar.gz \
#    && mkdir -p swoole \
#    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
#    && rm swoole.tar.gz \
#    && ( \
#    cd swoole \
#    && phpize \
#    && ./configure --enable-async-redis --enable-mysqlnd --enable-openssl --enable-http2 \
#    # 支持sockets资源
#    #&& --enable-sockets
#    && make -j$(nproc) \
#    && make install \
#    ) \
#    && rm -r swoole \
#    && docker-php-ext-enable swoole

# Swoole extension
RUN curl -L -o /tmp/swoole.tar https://pecl.php.net/get/swoole-${SWOOLE_VERSION}.tar \
    && tar xvf /tmp/swoole.tar \
    && rm -r /tmp/swoole.tar \
    && mkdir -p /usr/src/php/ext \
    && mv swoole-${SWOOLE_VERSION} /usr/src/php/ext/swoole \
    && docker-php-ext-install swoole

#RUN pecl install -y swoole-4.5.2 \
#    && docker-php-ext-enable swoole \
#    #&& php --ri swoole


# install composer
#RUN curl -sS https://getcomposer.org/installer \
#    | php --install-dir=/usr/local/bin --filename=composer

RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer self-update --clean-backups \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
    # && composer config -g repo.packagist composer https://packagist.phpcomposer.com


# librdkafka
RUN wget https://github.com/edenhill/librdkafka/archive/master.zip \
    && mv master.zip librdkafka-master.zip \
    && unzip librdkafka-master.zip \
    ###删除压缩包
    && rm -r librdkafka-master.zip \
    && cd librdkafka-master \
    && ./configure \
    && make \
    && make install

# php-rdkafka
RUN wget https://github.com/arnaud-lb/php-rdkafka/archive/$PHP_RDKAFKA_VERSION.tar.gz -O php-rdkafka-$PHP_RDKAFKA_VERSION.tar.gz \
    && tar -zxvf php-rdkafka-$PHP_RDKAFKA_VERSION.tar.gz \
    ###删除压缩包
    && rm -r php-rdkafka-$PHP_RDKAFKA_VERSION.tar.gz \
    && cd php-rdkafka-$PHP_RDKAFKA_VERSION \
    && phpize \
    && ./configure --with-php-config=php-config \
    && make \
    && make install


# ImageMagick(linux扩展)(pdf转png)
#RUN wget http://www.imagemagick.org/download/ImageMagick.tar.gz \
#    && tar zxvf ImageMagick.tar.gz \
#    && ls -la \
#    && cd ImageMagick-7.0.10-26 \
#    && ./configure --prefix=/usr/local/imagemagick \
#    && make \
#    && make install
    #&& export PKG_CONFIG_PATH=/usr/local/imagemagick/lib/pkgconfig/

# Imagick(php扩展)(pdf转png)
#RUN wget http://pecl.php.net/get/imagick-3.4.4.tgz \
#    && tar zxvf imagick-3.4.4.tgz \
#    #&& cd imagick-3.0.1 \
#    && cd imagick-3.4.4 \
#    && ls -la \
#    #&& /usr/bin/phpize \
#    && phpize \
#    #&& ./configure --with-php-config=/usr/bin/php-config --with-imagick=/usr/local/imagemagick \
#    && ./configure --with-php-config=php-config --with-imagick=/usr/local/imagemagick \
#    && make \
#    && make install


# xvfb(linux扩展-html转pdf)
#RUN apt-get upgrade && apt-get install -y \
#RUN apt-get install -y \
#    xvfb

# wkhtmltopdf(linux扩展-html转pdf)
#RUN wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
#    && tar xvfJ wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
#    && cp ./wkhtmltox/bin/wkhtmltopdf /usr/bin/wkhtmltopdf \
#    && chmod +x /usr/bin/wkhtmltopdf \
#    && cp /usr/bin/wkhtmltopdf /usr/local/bin/wkhtmltopdf

###可用,end



# sonar-scanner(代码质量检测管理工具)
#RUN wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip \
#    && unzip sonar-scanner-cli-3.3.0.1492-linux.zip && mv sonar-scanner-3.3.0.1492-linux/ sonar-scanner/ \
#    && cd sonar-scanner && cp ./bin/sonar-scanner /usr/local/bin/sonar-scanner
    #&& mv sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner