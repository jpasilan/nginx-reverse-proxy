FROM php:7.4-bullseye

ARG NODE_VERSION=16
WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates libzip-dev zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 \
       libmemcached-dev zlib1g-dev libpq-dev libc-client-dev libkrb5-dev libxml2-dev libldap2-dev libpcre3-dev \
    && mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && pecl install memcached pcov msgpack igbinary redis \
    && docker-php-ext-install gd json opcache \
       pdo \
       zip bcmath soap intl xmlrpc \
    && docker-php-ext-enable gd memcached opcache \
       pdo \
       zip bcmath soap intl pcov msgpack \
       igbinary redis xmlrpc \
    && php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt install -y nodejs \
    && npm install -g npm \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | tee /usr/share/keyrings/yarnkey.gpg >/dev/null \
    && echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y yarn \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Allow non-root users access to privileged ports.
RUN setcap "cap_net_bind_service=+ep" /usr/local/bin/php

RUN id www-user || useradd -ms /bin/bash --no-user-group -u 1337 www-user

ENV COMPOSER_HOME=/.composer
RUN mkdir /.composer \
    && chmod ugo+rw /.composer
COPY composer/config.json /.composer/config.json

COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/7.4/cli/conf.d/99-laravel.ini
RUN chmod +x /usr/local/bin/start-container

EXPOSE 80

ENTRYPOINT ["start-container"]
