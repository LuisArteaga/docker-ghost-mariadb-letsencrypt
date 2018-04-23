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
version: "2"

services:

  ghost:
    container_name: ${CONTAINER_GHOST_NAME}
    image: ghost:1.22.2
    volumes:
      - ${GHOST_APPS}:/var/lib/ghost/content/apps
      - ${GHOST_DATA}:/var/lib/ghost/content/data
      - ${GHOST_IMAGES}:/var/lib/ghost/content/images
      - ${GHOST_THEMES}:/var/lib/ghost/content/themes
    environment:
      - url=${GHOST_URL}
      - database__client=${DB_CLIENT}
      - database__connection__host=${CONTAINER_DB_NAME}
      - database__connection__user=${DB_USER}
      - database__connection__password=${DB_PASSWORD}
      - database__connection__database=${DB_NAME}
      - NODE_ENV=${STATUS}
      - VIRTUAL_HOST=${DOMAINS}
      - VIRTUAL_PORT=${PORT}
      - LETSENCRYPT_HOST=${DOMAINS}
      - LETSENCRYPT_EMAIL=${LETSENCRYPT_EMAIL}
    depends_on:
      - db
    restart: unless-stopped

  db:
    container_name: ${CONTAINER_DB_NAME}
    image: mariadb:10.2.14
    volumes:
      - ${DB_PATH}:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}
      - MYSQL_DATABASE=${DB_NAME}
      - MYSQL_USER=${DB_USER}
      - MYSQL_PASSWORD=${DB_PASSWORD}
    restart: unless-stopped

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
DB_PATH=/path-to-your/mysql

# Database Client
DB_CLIENT=mysql

# Root password for your database
DB_ROOT_PASSWORD=rootpassword

# Database name, user and password for your ghost
DB_NAME=databasename
DB_USER=username
DB_PASSWORD=userpassword

#
# Ghost Container configuration
#
CONTAINER_GHOST_NAME=ghost

# Path to store your ghost files
GHOST_APPS=/path-to-your/ghost/content/apps
GHOST_DATA=/path-to-your/ghost/content/data
GHOST_IMAGES=/path-to-your/ghost/content/images
GHOST_THEMES=/path-to-your/ghost/content/themes

# Your port for ghost
PORT=2368

# Your domain (or domains)
DOMAINS=domain.com,www.domain.com

# Your email for Let's Encrypt register
LETSENCRYPT_EMAIL=your_email@domain.com

# Your complete Site-URL (non-www or www)
GHOST_URL=https://www.domain.com

# Your Status of your environment (development or production)
STATUS=production
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
3. [@JrCs](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion)
4. [@evertramos](https://github.com/evertramos/docker-compose-letsencrypt-nginx-proxy-companion)
5. [@autoize](https://autoize.com/docker-compose-file-ghost/)