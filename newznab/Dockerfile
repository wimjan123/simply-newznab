FROM ubuntu:20.04
MAINTAINER thewillonline
RUN DEBIAN_FRONTEND="noninteractive"

ENV TZ='Europe/Amsterdam' \
    NNUSER="svnplus" \
    NNPASS="svnplu5"

RUN echo $TZ > /etc/timezone && \
    apt-get update && apt-get install -y tzdata && \
    rm /etc/localtime && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get clean

RUN apt update -y && \
    apt upgrade -y && \
    apt install -y \
    ssh screen apache2 php php-fpm && \
    apt install -y \
    php-pear php-gd php-mysql php-redis php-curl php-json php-mbstring wget lame \
    mediainfo subversion ffmpeg redis

#Install Rar & Unrar 6
RUN mkdir /rarinstall && \
  cd /rarinstall && \
  wget https://www.rarlab.com/rar/rarlinux-x64-6.0.0.tar.gz && \
  tar xfv rarlinux* && \
  cd rar && \
  cp unrar /usr/bin/ && cp rar /usr/bin && \
  rm /rarinstall -r

RUN mkdir /var/www/newznab && \
    mkdir /var/www/newznab/logs

RUN svn export --no-auth-cache --force --username svnplus --password svnplu5 svn://svn.newznab.com/nn/branches/nnplus /var/www/newznab/


RUN rm /var/www/newznab/*.txt && \
    rm -rf /var/www/newznab/www/install && \
    rm -rf /var/www/newznab/www/lib/smarty/templates_c/*

COPY config.php /var/www/newznab/www/config.php
RUN chmod 777 /var/www/newznab/www/config.php

COPY config.php /var/www/newznab/misc/update_scripts/
RUN chmod 777 /var/www/newznab/misc/update_scripts/
#
# COPY automated.config.php /var/www/newznab/www/automated.config.php
# RUN chmod 777 /var/www/newznab/www/automated.config.php
#
# COPY automated.config.php /var/www/newznab/misc/update_scripts/
# RUN chmod 777 /var/www/newznab/misc/update_scripts/

# RUN cd /var/www/newznab/ && \
#     php misc/update_scripts/update_database_version.php && \
#     cd /var/www/newznab/

COPY newznab.conf /etc/apache2/sites-available/newznab.conf

RUN service restart memcached && \
    service restart apache2 && \
    service restart php7.4-fpm

RUN chmod +x /var/www/newznab/svn.sh && \
    ./var/www/newznab/svn.sh

RUN chmod 777 /var/www/newznab/www/lib/smarty/templates_c && \
    chmod 777 /var/www/newznab/www/covers/movies && \
    chmod 777 /var/www/newznab/www/covers/anime && \
    chmod 777 /var/www/newznab/www/covers/music && \
    chmod 777 /var/www/newznab/www/covers/tv && \
    chmod 777 /var/www/newznab/www && \
    chmod 777 /var/www/newznab/www/install && \
    chmod 777 /var/www/newznab/db/cache && \
    chmod -R 777 /var/www/newznab/nzbfiles/


RUN sed -ri 's/(max_execution_time =) ([0-9]+)/\1 120/' /etc/php/7.4/fpm/php.ini && \
    sed -ri "s/(memory_limit =) (.*$)/\1 -1/" /etc/php/7.4/fpm/php.ini && \
    sed -ri 's/;(date.timezone =)/\1 Europe\/Amsterdam/' /etc/php/7.4/fpm/php.ini


RUN a2enmod proxy_fcgi setenvif && \
a2enconf php7.4-fpm && \
a2enmod rewrite && \
service restart php7.4-fpm && \
service restart apache2


COPY entrypoint.sh /entrypoint.sh
RUN chmod u+x /entrypoint.sh

EXPOSE 8800

VOLUME /var/www/newznab/nzbfiles
VOLUME /var/www/newznab/www/covers


ENTRYPOINT ["/entrypoint.sh"]
