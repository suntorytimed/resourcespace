# resourcespace-docker

[![Docker Pulls](https://img.shields.io/docker/pulls/suntorytimed/resourcespace?style=flat-square)](https://hub.docker.com/r/suntorytimed/resourcespace)

A docker image for ResourceSpace based on Ubuntu 20.04 LTS including OpenCV, poppler and php7.4. Older versions of ResourceSpace (9.2 and 9.3) are still based on 18.04 LTS and don't include poppler but xpdf. They also use php7.2.

Please report any issues on GitHub: https://github.com/suntorytimed/resourcespace-docker/issues

Starting with ResourceSpace 9.4 the containers are going to be based on Ubuntu:latest (currently 20.04 LTS), as I tested poppler as a proper xpdf replacement - which isn't shipped in 20.04 anymore - in ResourceSpace.

## docker-compose example

In this example I use the pre-existing nginx proxy from my Nextcloud instance which also includes a Let'sEncrypt compantion to enforce https.

```Dockerfile
version: "2"

# frontend network for resourcespace using the already existing nginx proxy of Nextcloud
# backend network without public accessibility for the database connection
networks:
  frontend:
    external:
      name: fpm_proxy-tier
  backend:
  
# for some reason using a local mountpoint results in an HTTP error 500
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
      - mariadb
    volumes:
      - include:/var/www/resourcespace/include
      - filestore:/var/www/resourcespace/filestore
    # variables for setting up https via the Let'sEncrypt companion
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
  
  mariadb:
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
