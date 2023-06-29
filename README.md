# Nginx Reverse Proxy

This repository is a proof-of-concept on running a reverse proxy for Docker container apps. This uses [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) which includes [docker-gen](https://github.com/nginx-proxy/docker-gen). With docker-gen, it makes it possible to decouple container apps that it serves.

Aside from the Docker Compose configuration needed for nginx-proxy, Dockerfiles as well as relevant system and software configurations are also present in this repository to create the container images for the following target languages:

- PHP 7
- PHP 8

## Network

For nginx-proxy to effectively serve as reverse proxy and forward HTTP requests to container apps, it must share the same Docker network with these apps. Prior to running `docker compose up`, create the network by running the script:

```
./bin/create-network
```

See the script for the actual Docker command and the name of the network that will be created. Running `docker compose up` without this network will result in an error.

## Container Apps

For docker-gen to detect container apps that will be proxied by nginx-proxy, the following environment variables must be defined in Docker Compose or via the `docker -e` command.

- VIRTUAL_HOST
- NETWORK_ACCESS=internal
- WWWUSER=${WWWUSER}
- WWWGROUP=${WWWGROUP}

The `VIRTUAL_HOST` is assigned the app's public domain name or URL. `NETWORK_ACCESS` should always be `internal` to make the apps accessible only within the Docker network. The `WWWUSER` and `WWWGROUP` are needed to correctly map the container and host users and groups to avoid file permission errors. The apps should also use the same Docker network created by the script above.

For reference, this [repository](https://github.com/jpasilan/nginx-reverse-proxy-apps) contains sample PHP apps configured to use this reverse proxy.

## Helper Script

There is a helper script that should be used in running container app commands like `composer` or `npm`. This script ensures that the app commands are run with the correct user permissions as this sets the `WWWUSER` and `WWWGROUP` environment variables before the app command is executed in the container.

```
./bin/docker-app
```

For example, to run `npm install` on the container, the script can be run like so:

```
./bin/docker-app npm install
```

Things to note about the script:

- The Docker container app is up and running.
- The script is executed within the app directory. That is, at the same level as `docker-compose.yml`

For convenience, it is recommended to alias this script in `.bashrc` for example or make it globally available by copying to any user-accessible `bin` directory in the host machine like `/usr/local/bin`.
