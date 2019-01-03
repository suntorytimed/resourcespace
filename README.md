# resourcespace-docker

## docker-compose example

In this example I use the pre-existing nginx proxy from my Nextcloud instance which also includes a Let'sEncrypt compantion to enforce https.

  version: "2"

  # frontend network for resourcespace using the already existing nginx proxy of Nextcloud
  # backend network without public accessibility for the database connection
  networks:
    frontend:
      external:
        name: fpm_proxy-tier
    backend:
  
  # docker volume for the database to make it stateful
  # can be replaced with a local mount instead
  volumes:
    mariadb:
  
  services:
    resourcespace:
      image: suntorytimed/resourcespace:latest
      restart: unless-stopped
      # links resourcespace to mariadb container and makes it accessible via the URL mariadb
      depends_on:
        - mariadb
      # local mounts to make it stateful
      volumes:
        - ./include:/var/www/resourcespace/include
        - ./filestore:/var/www/resourcespace/filestore
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
      # using a docker volume instead of a local mount
      volumes:
        - mariadb:/var/lib/mysql
      # only accesible in private network
      networks:
        - backend
