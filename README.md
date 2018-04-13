# Using Ghost with MariaDB with SSL enabled integrated with NGINX proxy and autorenew LetsEncrypt certificates

![docker-ghost-mariadb-letsencrypt](https://raw.githubusercontent.com/LuisArteaga/images/master/ghost.jpg)

This docker-compose should be used with WebProxy (the NGINX Proxy):

[https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion](https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion)


## Usage

After everything is settle, and you have your three containers running (_proxy, generator and letsencrypt_) you do the following:

1. Clone this repository:

```bash
git clone https://github.com/LuisArteaga/docker-ghost-mariadb-letsencrypt.git
```

Or just copy the content of `docker-compose.yml` and the `Dockerfile`, as of below:

```bash
version: '3'

services:
   db:
     container_name: ${CONTAINER_DB_NAME}
     image: mariadb:latest
     restart: unless-stopped
     volumes:
        - ${DB_PATH}:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
       MYSQL_DATABASE: ${MYSQL_DATABASE}
       MYSQL_USER: ${MYSQL_USER}
       MYSQL_PASSWORD: ${MYSQL_PASSWORD}

   ghost:
     depends_on:
       - db
     container_name: ${CONTAINER_GHOST_NAME}
     image: ghost:latest
     restart: unless-stopped
     volumes:
       - ${GHOST_CORE}:/var/www/html
       - ${GHOST_CONTENT}:/var/www/html/ghost/content
     environment:
       GHOST_DB_HOST: ${CONTAINER_DB_NAME}:3306
       GHOST_DB_NAME: ${MYSQL_DATABASE}
       GHOST_DB_USER: ${MYSQL_USER}
       GHOST_DB_PASSWORD: ${MYSQL_PASSWORD}
       GHOST_TABLE_PREFIX: ${GHOST_TABLE_PREFIX}
       VIRTUAL_HOST: ${DOMAINS}
       LETSENCRYPT_HOST: ${DOMAINS}
       LETSENCRYPT_EMAIL: ${LETSENCRYPT_EMAIL} 

networks:
    default:
       external:
         name: ${NETWORK}
```

2. Make a copy of our .env.sample and rename it to .env:

Update this file with your preferences.

```bash
# .env file to set up your ghost site

#
# Network name
# 
# Your container app must use a network conencted to your webproxy 
# https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion
#
NETWORK=webproxy

#
# Database Container configuration
# We recommend MySQL or MariaDB - please update docker-compose file if needed.
#
CONTAINER_DB_NAME=db

# Path to store your database
DB_PATH=/path/to/your/local/database/folder

# Root password for your database
MYSQL_ROOT_PASSWORD=root_password

# Database name, user and password for your ghost
MYSQL_DATABASE=database_name
MYSQL_USER=user_name
MYSQL_PASSWORD=user_password

#
# Ghost Container configuration
#
CONTAINER_GHOST_NAME=ghost

# Path to store your ghost files
GHOST_CORE=/path/to/your/ghost/core/files
GHOST_CONTENT=/path/to/your/ghost/content

# Table prefix
GHOST_TABLE_PREFIX=ghost_

# Your domain (or domains)
DOMAINS=domain.com,www.domain.com

# Your email for Let's Encrypt register
LETSENCRYPT_EMAIL=your_email@domain.com
```

>This container must use a network connected to your webproxy or the same network of your webproxy.

3. Start your project

```bash
docker-compose up -d
```

**Be patient** - when you first run a container to get new certificates, it may take a few minutes.

## WebProxy

[WebProxy - docker-compose-letsencrypt-nginx-proxy-companion](https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion)

----

## Issues

Please be advised that if are running docker on azure servers you must mount your database in your disks partitions (exemple: `/mnt/data/`) so your db container can work. This is a some kind of issue regarding Hyper-V sharing drivers... not really sure why.


## Full Source

1. [@jwilder](https://github.com/jwilder/nginx-proxy)
2. [@jwilder](https://github.com/jwilder/docker-gen)
3. [@JrCs](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion).