FROM ubuntu:latest
MAINTAINER Stefan Weiberg <sweiberg@suse.com>
ENV DEBIAN_FRONTEND="noninteractive"
RUN apt-get update && apt-get install -y \
    vim \
    imagemagick \
    apache2 \
    subversion \
    ghostscript \
    antiword \
    xpdf \
    libimage-exiftool-perl \
    cron \
    postfix \
    wget \
    php \
    php-dev \
    php-gd \
    php-mysqlnd \
    php-mbstring \
    php-zip \
    libapache2-mod-php \
    ffmpeg \
    libopencv-dev \
    python-opencv \
    python3 \
    python3-pip \
 && rm -rf /var/lib/apt/lists/*
RUN sed -i -e "s/upload_max_filesize\s*=\s*2M/upload_max_filesize = 10G/g" /etc/php/7.2/apache2/php.ini                                                      
RUN sed -i -e "s/post_max_size\s*=\s*8M/post_max_size = 10G/g" /etc/php/7.2/apache2/php.ini                                                                  
RUN sed -i -e "s/max_execution_time\s*=\s*30/max_execution_time = 10000/g" /etc/php/7.2/apache2/php.ini                                                      
RUN sed -i -e "s/memory_limit\s*=\s*128M/memory_limit = 4G/g" /etc/php/7.2/apache2/php.ini

RUN printf '<Directory /var/www/>\n\
\tOptions FollowSymLinks\n\
</Directory>\n'\
>> /etc/apache2/sites-enabled/000-default.conf

ADD cronjob /etc/cron.daily/resourcespace

WORKDIR /var/www/html
RUN rm index.html
RUN svn co https://svn.resourcespace.com/svn/rs/releases/9.1 .
RUN mkdir filestore
RUN chmod 777 filestore
RUN chmod -R 777 include/
CMD apachectl -D FOREGROUND
