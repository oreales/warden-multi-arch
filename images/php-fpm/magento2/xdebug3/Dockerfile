ARG PHP_TAG
FROM ghcr.io/oreales/warden-php-m2:${PHP_TAG}

USER root

RUN install-php-extensions xdebug

COPY xdebug3/etc/*.ini /etc/
COPY xdebug3/etc/xdebug-config /usr/local/etc/php/conf.d/xdebug-config

RUN cat /usr/local/etc/php/conf.d/xdebug-config >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && rm /usr/local/etc/php/conf.d/xdebug-config

USER www-data
