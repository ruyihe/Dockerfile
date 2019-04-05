FROM php:5.6.39-fpm-alpine3.8

MAINTAINER heruyi@shopex.cn

ARG TIMEZONE=Asia/Shanghai

RUN set -xe \
# init alpine repo
    && { \
      echo "https://mirrors.huaweicloud.com/alpine/v3.8/main"; \
      echo "https://mirrors.huaweicloud.com/alpine/v3.8/community"; \
    } > /etc/apk/repositories \
# temporary resolve solution
    && sed -i 's/26:"https/25:"http/g' /usr/local/lib/php/.channels/pecl.php.net.reg \
# Install phpize deps
    && docker-php-source extract \
    && apk add --no-cache --update --virtual .phpize-deps $PHPIZE_DEPS \

# Install gd
    && apk add --no-cache --update --virtual .gd-deps libpng-dev libjpeg-turbo-dev libwebp-dev freetype-dev \
    && docker-php-ext-configure gd \
        --with-jpeg-dir=/usr/include \
        --with-png-dir=/usr/include \
        --with-webp-dir=/usr/include \
        --with-freetype-dir=/usr/include \
    && docker-php-ext-install gd \

# Set time zone
    && apk add --no-cache --update --virtual .timezone-deps tzdata \
    && cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \

# Install bcmath
    && docker-php-ext-install bcmath \

# Install mcrypt
    && apk add --no-cache --virtual .mcrypt-deps libmcrypt-dev \
    && docker-php-ext-install mcrypt \

# Install mysql
    && docker-php-ext-install mysql \
    && docker-php-ext-install mysqli \
    && docker-php-ext-install pdo_mysql \

# Install redis
    && pecl install redis-4.2.0 \
    && docker-php-ext-enable redis \

# Install mongo
    && apk add --no-cache --virtual .mongo-deps openssl-dev \
    && pecl install mongo-1.6.16 \
    && docker-php-ext-enable mongo \

# Install memcache
    && pecl install memcache-2.2.7 \
    && docker-php-ext-enable memcache \

# Install zip
    && apk add --no-cache --virtual .zip-deps libzip-dev \
    && pecl install zip-1.15.4 \
    && docker-php-ext-enable zip \

# Install php extensions dependence
    && runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )" \
    && apk add --virtual .phpexts-rundeps $runDeps \
# Clean
    && docker-php-source delete \
    && apk del --purge .gd-deps .phpize-deps .timezone-deps .mcrypt-deps .mongo-deps .zip-deps \
    && rm -rf /var/cache/apk/* \
    && rm -rf /tmp/*

COPY ./php.ini "$PHP_INI_DIR/php.ini"