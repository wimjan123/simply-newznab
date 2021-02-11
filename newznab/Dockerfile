FROM ubuntu:20.04
MAINTAINER Fekhoo <fekhoo@fekhoo.net>
ARG DEBIAN_FRONTEND=noninteractive

# Add Variables
ENV NNUSER="svnplus" \
    NNPASS="svnplu5" \
    TZ="Europe/Amsterdam"

RUN echo $TZ > /etc/timezone && \
    apt-get update && apt-get install -y tzdata && \
    rm /etc/localtime && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure -f noninteractive tzdata && \
    apt-get clean

# Install required packages
RUN apt-get -y update && \
    apt-get install -yq ssh screen tmux apache2 php php-fpm php-dev \
    php-pear php-gd php-mysql php-memcache php-curl php-json php-mbstring \
    lame mediainfo subversion ffmpeg memcached nano wget

#Install Rar & Unrar 6
RUN mkdir /rarinstall && \
    cd /rarinstall && \
    wget https://www.rarlab.com/rar/rarlinux-x64-6.0.0.tar.gz && \
    tar xfv rarlinux* && \
    cd rar && \
    cp unrar /usr/bin/ && cp rar /usr/bin && \
    rm /rarinstall -r


# Creating Newznab Folders from SVN
RUN mkdir /var/www/newznab/ && \
    svn export --no-auth-cache --force --username $NNUSER --password $NNPASS svn://svn.newznab.com/nn/branches/nnplus /var/www/newznab/ && \
    chmod 777 /var/www/newznab/www/lib/smarty/templates_c && \
    chmod 777 /var/www/newznab/www/covers/movies && \
    chmod 777 /var/www/newznab/www/covers/anime  && \
    chmod 777 /var/www/newznab/www/covers/music  && \
    chmod 777 /var/www/newznab/www/covers/tv && \
    chmod 777 /var/www/newznab/www  && \
    chmod 777 /var/www/newznab/www/install  && \
    chmod -R 777 /var/www/newznab/nzbfiles && \
    chmod -R 777 /var/www/newznab/www/covers

#Add Newznab Config File
COPY config.php /var/www/newznab/www/config.php
RUN chmod 777 /var/www/newznab/www/config.php

# Configure Apache for Newznab site
COPY newznab.conf /etc/apache2/sites-available/newznab.conf

RUN mkdir /var/www/newznab/logs/

#Add newznab processing & Config script
COPY entrypoint.sh /entrypoint.sh
RUN chmod u+x /entrypoint.sh

EXPOSE 80

VOLUME /var/www/newznab/nzbfiles
VOLUME /var/www/newznab/www/covers

RUN mkdir -pv /opt/sphinx/log /opt/sphinx/index
VOLUME /opt/sphinx/index

# http://sphinxsearch.com/files/sphinx-3.0.3-facc3fb-linux-amd64.tar.gz
RUN wget http://sphinxsearch.com/files/sphinx-${SPHINX_VERSION}-linux-amd64.tar.gz -O /tmp/sphinxsearch.tar.gz
RUN cd /opt/sphinx && tar -xf /tmp/sphinxsearch.tar.gz
RUN rm /tmp/sphinxsearch.tar.gz

# point to sphinx binaries
ENV PATH "${PATH}:/opt/sphinx/sphinx-3.0.3/bin"
RUN indexer -v

# redirect logs to stdout
RUN ln -sv /dev/stdout /opt/sphinx/log/query.log
RUN ln -sv /dev/stdout /opt/sphinx/log/searchd.log

# expose TCP port
EXPOSE 36307

VOLUME /opt/sphinx/conf

ENTRYPOINT ["/entrypoint.sh"]
