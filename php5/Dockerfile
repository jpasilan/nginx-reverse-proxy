FROM php:5.6-fpm

ARG NODE_VERSION=16

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Update the APT sources for Debian Stretch.
RUN echo "deb [trusted=yes] http://archive.debian.org/debian stretch main non-free contrib" > /etc/apt/sources.list \
    && echo 'deb-src [trusted=yes] http://archive.debian.org/debian/ stretch main non-free contrib' >> /etc/apt/sources.list \
    && echo 'deb [trusted=yes] http://archive.debian.org/debian-security/ stretch/updates main non-free contrib' >> /etc/apt/sources.list

RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg gosu curl ca-certificates libzip-dev zip unzip \
       supervisor libcap2-bin libpng-dev libmemcached-dev zlib1g-dev libpq-dev libc-client-dev \
       libkrb5-dev libxml2-dev libldap2-dev libpcre3-dev apt-transport-https wget \
       freetds-bin freetds-dev freetds-common

# This is needed to compile PDO_DBLIB. See https://docs.moodle.org/31/en/Installing_MSSQL_for_PHP
RUN dpkg -L freetds-dev | grep libsybdb.a | xargs -i ln -s {} /usr/lib

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/
RUN install-php-extensions @fix_letsencrypt \
    && pecl install memcached-2.2.0 msgpack-0.5.7 igbinary-2.0.8 redis-2.2.8 \
    && docker-php-ext-install gd json opcache \
       pdo pdo_dblib mssql zip bcmath soap intl xmlrpc \
    && docker-php-ext-enable gd opcache \
       pdo pdo_dblib mssql zip bcmath soap intl xmlrpc \
       memcached.so msgpack.so igbinary.so redis.so

# Install other software packages then clean up.
RUN echo 'deb http://nginx.org/packages/mainline/debian/ stretch nginx' > /etc/apt/sources.list.d/nginx.list \
    && echo 'deb-src http://nginx.org/packages/mainline/debian/ stretch nginx' >> /etc/apt/sources.list.d/nginx.list \
    && wget -qO - https://nginx.org/keys/nginx_signing.key | apt-key add - \
    && echo "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" > /etc/apt/preferences.d/99nginx

RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_VERSION.x nodistro main" \
      | tee /etc/apt/sources.list.d/nodesource.list

RUN php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && apt-get update \
    && apt install -y --no-install-recommends nodejs \
    && apt install -y --no-install-recommends nginx \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV COMPOSER_HOME=/.composer
RUN mkdir /.composer && chmod ugo+rw /.composer
COPY common/composer/config.json /.composer/config.json

RUN groupadd --force -g 1337 www-user
RUN useradd -ms /bin/bash --no-user-group -g www-user -u 1337 www-user

# PHP related configurations
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"
COPY common/php-conf/php.ini "$PHP_INI_DIR/conf.d/99-www-user.ini"
# For php-fpm, overwrite zz-docker.conf instead of www.conf
COPY common/php-conf/zz-docker.conf /usr/local/etc/php-fpm.d/zz-docker.conf

# nginx 1.1x related configurations
COPY common/nginx/nginx.conf /etc/nginx/nginx.conf
COPY common/nginx/nginx-default.conf /etc/nginx/conf.d/default.conf

COPY common/start-container /usr/local/bin/start-container
COPY common/php-conf/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN chmod +x /usr/local/bin/start-container

EXPOSE 80

ENTRYPOINT ["start-container"]

