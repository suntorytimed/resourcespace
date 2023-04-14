# resourcespace-docker

[![Docker Pulls](https://img.shields.io/docker/pulls/suntorytimed/resourcespace?style=flat-square)](https://hub.docker.com/r/suntorytimed/resourcespace)

A docker image for ResourceSpace based on Ubuntu:latest container including OpenCV, poppler and php8.1.

Please report any issues on GitHub: https://github.com/suntorytimed/resourcespace-docker/issues

## docker-compose example

In this example I use a pre-existing nginx proxy using the [nginx-proxy/acme-companion](https://github.com/nginx-proxy/acme-companion) container, which includes a Let's Encrypt/ACME companion to enforce a properly signed https. Please consult the documentation of that container for setup instructions.

```Dockerfile
version: "2"

# frontend network for resourcespace using an already existing nginx proxy for Let's Encrypt
# backend network without public accessibility for the database connection
networks:
  frontend:
    external:
      name: acme-proxy-network
  backend:

# Trying to use bind volumes directly resulted in a 500 error, but using named volumes worked  
volumes:
  mariadb:
  include:
  filestore:

services:
  resourcespace:
    image: suntorytimed/resourcespace:latest
    restart: unless-stopped
    # links resourcespace to mariadb container and makes it accessible via the URL mariadb
    depends_on:
      - mariadb-resourcespace
    volumes:
      - include:/var/www/resourcespace/include
      - filestore:/var/www/resourcespace/filestore
    # variables for setting up https via the Let's Encrypt/ACME companion
    environment:
      - HOSTNAME=dam.example.com
      - VIRTUAL_HOST=dam.example.com
      - VIRTUAL_PORT=80
      - LETSENCRYPT_HOST=dam.example.com
      - LETSENCRYPT_EMAIL=admin@example.com
    # public and private network
    networks:
      - frontend
      - backend
    # no port bind necessary due to the nginx proxy
    expose:
      - 80
  
  mariadb-resourcespace:
    image: mariadb
    restart: unless-stopped
    # env file containing the db configuration
    env_file:
      - db.env
    volumes:
      - mariadb:/var/lib/mysql
    # only accesible in private network
    networks:
      - backend
```

When using this example, you need to enter `mariadb-resourcespace` as the URL for the database in the initial ResourceSpace setup. You will also have to provide a db.env file in the same folder as your docker-compose.yml:

```
MYSQL_PASSWORD=secure-password
MYSQL_ROOT_PASSWORD=even-more-secure.password
MYSQL_DATABASE=resourcespace
MYSQL_USER=resourcespace
```

With this db.env file you set up your mariadb database for ResourceSpace. You will have to enter the login data in the initial ResourceSpace setup. The MySQL binary path in that setup needs to be empty, as ResourceSpace won't be using a local mysql/mariadb installation.
