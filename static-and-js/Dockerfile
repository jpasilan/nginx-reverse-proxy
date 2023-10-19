FROM debian:bookworm-slim

ARG NODE_VERSION=18

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg2 gosu curl ca-certificates libzip-dev zip unzip \
       supervisor libcap2-bin apt-transport-https

# Install other software packages then clean up.
RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_VERSION.x nodistro main" \
      | tee /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt install -y --no-install-recommends nodejs \
    && apt install -y --no-install-recommends nginx \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN groupadd --force -g 1337 www-user
RUN useradd -ms /bin/bash --no-user-group -g www-user -u 1337 www-user

# nginx related configurations
COPY common/nginx/nginx.conf /etc/nginx/nginx.conf
COPY common/nginx/nginx-default-static.conf /etc/nginx/sites-available/default

COPY common/start-container /usr/local/bin/start-container
COPY common/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN chmod +x /usr/local/bin/start-container

EXPOSE 80

ENTRYPOINT ["start-container"]

