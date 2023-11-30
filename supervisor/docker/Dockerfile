FROM php:8.2-fpm-alpine

## Install libraries and reduce container size
RUN apk --no-cache add  \
    pcre-dev  \
    libsodium-dev \
    zlib-dev \
    libpng-dev \
    libzip-dev \
    ${PHPIZE_DEPS} \
    openrc \
    supervisor \
  && pecl install redis \
  && docker-php-ext-enable opcache redis.so \
  && apk del pcre-dev ${PHPIZE_DEPS}

## Copy composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

## Install postgresSQL
RUN apk --no-cache --update --repository https://dl-cdn.alpinelinux.org/alpine/v$ALPINE_VERSION/main/ add postgresql-dev
RUN docker-php-ext-configure pdo_pgsql

## Install php dependencies
RUN docker-php-ext-install pdo pdo_pgsql bcmath intl sodium zip gd

## Curl certificate
RUN wget https://curl.se/ca/cacert-2023-08-22.pem -O /usr/local/share/ca-certificates/cacert.crt
RUN update-ca-certificates

## Create a first copy of the source code
COPY . /app

## Copy supervisor config
COPY ./infra/docker/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

##start supervisor
CMD ["/usr/bin/supervisord","-n", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
