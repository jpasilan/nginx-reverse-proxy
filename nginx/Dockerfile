FROM nginx:1.25.0-alpine

# Install dependencies
RUN apk add --no-cache --virtual .run-deps bash ca-certificates openssl \
    && update-ca-certificates

# Configure nginx
RUN sed -i 's/worker_processes  1/worker_processes  auto/' /etc/nginx/nginx.conf \
    && sed -i 's/worker_connections  1024/worker_connections  10240/' /etc/nginx/nginx.conf \
    && mkdir -p '/etc/nginx/dhparam'

COPY ./nginx/dhparam/ffdhe4096.pem /etc/nginx/dhparam/dhparam.pem
COPY ./nginx/network_internal.conf /etc/nginx/

