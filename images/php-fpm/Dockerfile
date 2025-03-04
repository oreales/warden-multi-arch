ARG PHP_TAG
FROM ghcr.io/oreales/php-fpm-loaders:${PHP_TAG}
ARG PHP_TAG

# Clear undesired settings from base fpm images
ENV COMPOSER_ALLOW_SUPERUSER=
ENV COMPOSER_HOME=

ENV MAILHOG_HOST    mailhog
ENV MAILHOG_PORT    1025

RUN apt update

RUN if [ $PHP_TAG != "7.4-deb" ]  \
    && [ $PHP_TAG != "7.3-deb" ]; then \
        BIND_UTILS=bind9utils; \
    else \
        BIND_UTILS=bind9-utils; \
    fi \
    && if [ $PHP_TAG = "7.0-deb" ]; then \
        ACK=ack-grep \
        MARIADB_SERVER=; \
    else \
        ACK=ack \
        MARIADB_SERVER=mariadb-server; \
    fi \
    && apt install -y pv sudo ${BIND_UTILS} python3-pip bash-completion rsync socat oathtool \
        wget ${ACK} automake autoconf nano vim cron ${MARIADB_SERVER}

RUN if [ $PHP_TAG = "7.0-deb" ]; then \
        apt install -y python; \
    else \
        rm -rf /usr/bin/python && apt install -y python2 && ln -s /usr/bin/python2 /usr/bin/python; \
    fi

ENV NODE_VERSION    10
RUN npm install -g n \
    && n install ${NODE_VERSION} \
    && rm -rf /usr/local/n/versions/node

COPY docker-entrypoint /usr/local/bin/
COPY etc/profile.d/*.sh /etc/profile.d/
COPY etc/*.ini /etc/
COPY etc/opcache-config /usr/local/etc/php/conf.d/opcache-config
COPY etc/php.d/*.ini /usr/local/etc/php/conf.d
COPY etc/php.d/05-additions.ini.template /usr/local/etc/php/conf.d

RUN cat /usr/local/etc/php/conf.d/opcache-config >> /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini \
    && rm /usr/local/etc/php/conf.d/opcache-config

ARG TARGETARCH

# Install mhsendmail to support routing email through mailhog
RUN mkdir -p /tmp/mhsendmail \
    && cd /tmp/mhsendmail \
    && curl -vLOs https://github.com/drpayyne/mhsendmail/releases/download/v0.3.0/mhsendmail_linux_$TARGETARCH \
    && chmod +x mhsendmail_linux_$TARGETARCH \
    && mv mhsendmail_linux_$TARGETARCH /usr/local/bin/mhsendmail

# Install awscli to support data backfill workflows using S3 storage; segno for QR code generation
RUN if [ $PHP_TAG = "7.0-deb" ]; then \
        pip3 install awscli segno; \
    else \
        pip3 install awscli segno --no-cache-dir; \
    fi

# Configure www-data user as primary php-fpm user for better local dev experience
RUN chmod 0755 ~www-data \
    && mkdir -p /var/www/html \
    && chown www-data:www-data /var/www/html \
    && echo "www-data ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/nopasswd \
    && PHP_FPM_USER=$(grep -i '^user = ' /usr/local/etc/php-fpm.d/www.conf | awk '{print $3}') \
    && PHP_FPM_GROUP=$(grep -i '^group = ' /usr/local/etc/php-fpm.d/www.conf | awk '{print $3}') \
    && perl -pi -e 's/^(user|group) = php-fpm$/$1 = www-data/g' /usr/local/etc/php-fpm.d/www.conf

WORKDIR /var/www/html
USER www-data
