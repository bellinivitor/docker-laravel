FROM php:8.5-fpm-alpine

## Host user UID/GID, so files created in the volume get the right owner (relevant on Linux)
ARG HOST_UID=1000
ARG HOST_GID=1000

## Supervisor + tools used by Composer
RUN apk add --no-cache git unzip supervisor

## Install PHP extensions (the installer resolves and removes build dependencies on its own)
## On PHP 8.5 opcache is built in; on earlier versions add "opcache" to the list
COPY --from=mlocati/php-extension-installer:2 /usr/bin/install-php-extensions /usr/local/bin/
RUN install-php-extensions \
      bcmath \
      gd \
      intl \
      pcntl \
      pdo_pgsql \
      redis \
      zip

## Copy composer
COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

## Match www-data with the host user
RUN apk add --no-cache --virtual .user-deps shadow \
  && groupmod -o -g ${HOST_GID} www-data \
  && usermod -o -u ${HOST_UID} -g ${HOST_GID} www-data \
  && apk del .user-deps

WORKDIR /app

## Create a first copy of the source code (in dev the ./:/app volume overrides this copy)
COPY --chown=www-data:www-data . /app

## Copy supervisor config
COPY ./docker/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

## Start supervisor (manages php-fpm, workers and the scheduler)
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
