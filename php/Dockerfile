FROM daocloud.io/library/php:7.3-fpm-alpine

COPY config/php.ini /usr/local/etc/php

#RUN apk add autoconf gcc \
#    && wget https://github.com/phpredis/phpredis/archive/3.1.3.tar.gz \
#    && tar -zxvf 3.1.3.tar.gz \
#    && cd phpredis-3.1.3 \
#    && /usr/local/bin/phpize \
#    && ./configure --with-php-config=/usr/local/bin/php-config \
#    && make \
#    && make install \

ENV PHPREDIS_VERSION 5.0.2
ENV SWOOLE_VERSION 4.4.17
ENV RABBITMQ_AMQP 1.9.4

# 修改镜像源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    &&   apk update \
    &&   apk add ca-certificates wget \
    &&   update-ca-certificates

RUN cd ~ \
    && adduser -s /bin/bash git -D -H \
    && wget https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
    && tar -zxvf $PHPREDIS_VERSION.tar.gz \
    && mkdir -p /usr/src/php/ext \
    && mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis \
    && docker-php-ext-install redis \

    && apk add libstdc++ libpng-dev \
    && cd ~ \
    && wget https://github.com/swoole/swoole-src/archive/v$SWOOLE_VERSION.tar.gz \
    && tar -zxvf v$SWOOLE_VERSION.tar.gz \
    && mkdir -p /usr/src/php/ext \
    && mv swoole-src-$SWOOLE_VERSION /usr/src/php/ext/swoole \
    && docker-php-ext-install swoole \

    #&& apk add libtool autoconf automake cmake make libc-dev openssl-dev gcc \
    && apk add cmake make libc-dev openssl-dev gcc \
    && cd ~ \
    && wget https://github.com/alanxz/rabbitmq-c/archive/v0.9.0.tar.gz \
    && tar -zxvf v0.9.0.tar.gz \
    && cd rabbitmq-c-0.9.0 \
    && mkdir build \
    && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX=/usr/local/rabbitmq-c-0.9.0 .. \
    && cmake --build .  --target install \

    && cd ~ \
    && wget https://pecl.php.net/get/amqp-$RABBITMQ_AMQP.tgz \
    && tar -zxvf amqp-$RABBITMQ_AMQP.tgz \
    && mkdir -p /usr/src/php/ext \
    && mv amqp-$RABBITMQ_AMQP /usr/src/php/ext/amqp \
    && docker-php-ext-configure amqp --with-php-config=/usr/local/bin/php-config  --with-librabbitmq-dir=/usr/local/rabbitmq-c-0.9.0 \
    && mv /usr/local/rabbitmq-c-0.9.0/lib64 /usr/local/rabbitmq-c-0.9.0/lib \
    && docker-php-ext-install amqp \

    && docker-php-ext-install pdo_mysql mysqli pcntl sockets \
    
    && apk update \
    && apk add freetype freetype-dev libpng libpng-dev libjpeg-turbo libjpeg-turbo-dev \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && apk del freetype-dev libpng-dev libjpeg-turbo-dev \
    && rm /var/cache/apk/* \    

    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \
    && composer config -g repo.packagist composer https://packagist.phpcomposer.com \
