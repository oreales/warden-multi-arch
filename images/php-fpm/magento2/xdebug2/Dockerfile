ARG PHP_TAG
FROM ghcr.io/cvarela-usesi/warden-php-m2:${PHP_TAG}

USER root

RUN install-php-extensions xdebug

COPY xdebug2/etc/*.ini /etc/
COPY xdebug2/etc/php.d/*.ini /usr/local/etc/php/conf.d/

USER www-data
