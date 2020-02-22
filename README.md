# PHP Apps with Docker

Local Development setup for php apps. Wordpress (master branch)
and Laravel (In Progress)

# Installation
Requirements
- You need to have [Docker](https://docs.docker.com/engine/installation/) installed

Run in root folder,
~~~~
docker-compose build && docker-compose up -d
~~~~

Login to the container,
~~~~
docker exec -it server bash
~~~~

## Docker Stack
- Ubuntu 18.04
    - PHP 7.4
    - PHP 7.4 extensions and other tools & packages
     (Composer, wp-cli, xdebug, PHPUnit etc..). Check Dockerfile for more. 
- MySQL 5.7
- phpMyAdmin
- Redis 3.2
