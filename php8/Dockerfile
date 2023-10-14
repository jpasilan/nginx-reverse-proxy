FROM php:8.2-fpm-bullseye

ARG NODE_VERSION=18

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg2 gosu curl ca-certificates libzip-dev zip unzip \
       supervisor libcap2-bin libpng-dev libmemcached-dev zlib1g-dev libpq-dev libc-client-dev \
       libkrb5-dev libxml2-dev libldap2-dev libpcre3-dev apt-transport-https \
    && pecl install memcached pcov msgpack igbinary redis xmlrpc-1.0.0RC3 \
    && docker-php-ext-install gd opcache zip bcmath soap intl \
    && docker-php-ext-enable gd memcached opcache zip bcmath \
       soap intl pcov msgpack igbinary redis xmlrpc.so

# Install MSSQL extension
# Getting the source from Ubuntu instead of Debian since there seems to be an issue in the latter.
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
    && curl https://packages.microsoft.com/config/ubuntu/22.04/prod.list > /etc/apt/sources.list.d/mssql-release.list \
    && apt-get update \
    && ACCEPT_EULA=Y apt-get install -y --no-install-recommends msodbcsql17 unixodbc-dev

RUN pear config-set php_ini `php --ini | grep "Loaded Configuration" | sed -e "s|.*:\s*||"` system \
    && pecl install sqlsrv \
    && pecl install pdo_sqlsrv \
    && docker-php-ext-enable sqlsrv.so \
    && docker-php-ext-enable pdo_sqlsrv.so

# Install other software packages then clean up.
RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_VERSION.x nodistro main" \
      | tee /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt install -y --no-install-recommends nodejs

RUN php -r "readfile('https://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
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

# nginx related configurations
COPY common/nginx/nginx.conf /etc/nginx/nginx.conf
COPY common/nginx/nginx-default.conf /etc/nginx/sites-available/default

COPY common/start-container /usr/local/bin/start-container
COPY common/php-conf/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN chmod +x /usr/local/bin/start-container

EXPOSE 80

ENTRYPOINT ["start-container"]

