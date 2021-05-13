# Default Dockerfile
#
# @link     https://www.hyperf.io
# @document https://doc.hyperf.io
# @contact  group@hyperf.io
# @license  https://github.com/hyperf-cloud/hyperf/blob/master/LICENSE

FROM hyperf/hyperf:7.3-alpine-v3.11-swoole
LABEL maintainer="Hyperf Developers <group@hyperf.io>" version="1.0" license="MIT"

##
# ---------- env settings ----------
##
# --build-arg timezone=Asia/Shanghai
ARG timezone

ENV TIMEZONE=${timezone:-"Asia/Shanghai"} \
    COMPOSER_VERSION=2.0.13
    #APP_ENV=prod

# 修改镜像源

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    &&   apk update \
    &&   apk add --no-cache ca-certificates wget \
    &&   update-ca-certificates 2>/dev/null || true


# update
RUN set -ex \
    #&& apk update \
    # install composer
	#&& apk add --no-cache php7-simplexml php7-tokenizer php7-fileinfo php7-xmlwriter \
    && cd /tmp \
    && wget https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod u+x composer.phar \
    && mv composer.phar /usr/local/bin/composer \
	&& composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ \
    # show php version and extensions
    && php -v \
    && php -m \
    #  ---------- some config ----------
    && cd /etc/php7 \
    # - config PHP
    && { \
        echo "upload_max_filesize=100M"; \
        echo "post_max_size=108M"; \
        echo "memory_limit=1024M"; \
        echo "date.timezone=${TIMEZONE}"; \+
    } | tee conf.d/99-overrides.ini \
    # - config timezone
    && ln -sf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone \
    # ---------- clear works ----------
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n" \
	&& apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing gnu-libiconv

ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

#COPY . /opt/www

#WORKDIR /opt/www

#RUN composer install --no-dev -o

#EXPOSE 9501

#ENTRYPOINT ["php", "/opt/www/bin/hyperf.php", "start"]