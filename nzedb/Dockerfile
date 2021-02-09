FROM ubuntu:18.04
MAINTAINER thewillonline

# Configure Timezone
ENV TZ 'Europe/Amsterdam'
RUN echo $TZ > /etc/timezone && \
apt-get update && apt-get install -y tzdata && \
rm /etc/localtime && \
ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
dpkg-reconfigure -f noninteractive tzdata && \
apt-get clean

RUN apt-get update -y && \
    apt-get install -y curl systemd software-properties-common && \
    add-apt-repository -y ppa:ondrej/php && \
    apt-add-repository multiverse && \
    # apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc' && \
    # add-apt-repository 'deb [arch=amd64,arm64,ppc64el] https://mirror.zol.co.zw/mariadb/repo/10.5/ubuntu bionic main' && \
    apt-get update -y && \
    apt-get upgrade -y && \
    apt autoremove -y

#installing apache2
RUN apt-get install -y \
  apache2 apache2-utils

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
COPY . /var/www/html/
EXPOSE 80

RUN service apache2 start && \
  chown www-data /var/www/html/ -R

#Instaling MariaDB
RUN apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8 && \
  add-apt-repository -y 'deb [arch=amd64] http://mirror.zol.co.zw/mariadb/repo/10.3/ubuntu bionic main' && \
  apt update -y && \
  apt install -y mariadb-client

#mysql_secure_installation
# ADD https://github.com/wimjan123/simply-nzedb/blob/master/nzedb/automate_mysqlsecure.sh /tmp/
# RUN chmod +x /tmp/automate_mysqlsecure.sh && ./tmp/automate_mysqlsecure.sh

#PHP 7.2
RUN apt install -y \
  php7.2-fpm php7.2-mysql php7.2-common php7.2-gd php7.2-json php7.2-cli php7.2-curl libapache2-mod-php7.2 time && \
  a2enmod php7.2 && \
  service apache2 restart && \
  apt install -y

RUN usermod -a -G www-data root

#s6 installer
ADD https://github.com/just-containers/s6-overlay/releases/download/v2.2.0.1/s6-overlay-amd64-installer /tmp/
RUN chmod +x /tmp/s6-overlay-amd64-installer && ./tmp/s6-overlay-amd64-installer /

# vnstat in testing repo

# mytop + deps
RUN apt-get -y install \
  # mariadb-server  \
  perl \
  libmysqlclient-dev \
  libterm-readkey-perl

# Install composer
RUN curl https://getcomposer.org/installer | php7.2 -- --install-dir=/usr/bin --filename=composer --version=1.10.19

#install python and git and extra's
RUN apt-get install -y \
  python3-pip python3 \
  git php-imagick php-pear php7.2-curl php7.2-gd php7.2-json php7.2-dev php7.2-gd php7.2-mbstring php7.2-xml zip unzip \
  python python-setuptools python-dev build-essential \
  p7zip-full lame mediainfo ffmpeg nano wget

#Install Rar & Unrar 6
RUN mkdir /rarinstall && cd /rarinstall && \
  wget https://www.rarlab.com/rar/rarlinux-x64-6.0.0.tar.gz && \
  tar xfv rarlinux* && \
  cd rar && \
  cp unrar /usr/bin/

# Install Python MySQL Modules
RUN export LC_ALL="en_US.UTF-8"
RUN pip3 install --upgrade pip && \
  pip3 install --upgrade setuptools && \
  pip3 install cymysql pynntp

# Configure PHP
RUN printf "<VirtualHost *:80> \n\
    ServerAdmin webmaster@localhost \n\
    ServerName 127.0.0.1 \n\
    DocumentRoot "/var/www/nZEDb/www" \n\
    LogLevel warn \n\
    ServerSignature Off \n\
    ErrorLog /var/log/apache2/error.log \n\
    <Directory "/var/www/nZEDb/www"> \n\
       Options FollowSymLinks \n\
       AllowOverride All \n\
       Require all granted \n\
    </Directory> \n\
    Alias /covers /var/www/nZEDb/resources/covers \n\
</VirtualHost> \
" >> /etc/apache2/sites-available/nzedb.conf

RUN  a2dissite 000-default && \
 a2ensite nzedb.conf && \
 a2enmod rewrite && \
 service apache2 restart

RUN sed -ri 's/(max_execution_time =) ([0-9]+)/\1 120/' /etc/php/7.2/apache2/php.ini && \
  sed -ri "s/(memory_limit =) (.*$)/\1 -1/" /etc/php/7.2/apache2/php.ini && \
  sed -ri 's/;(date.timezone =)/\1 Europe\/Amsterdam/' /etc/php/7.2/apache2/php.ini && \
  sed -ri 's/(max_execution_time =) ([0-9]+)/\1 120/' /etc/php/7.2/cli/php.ini && \
  sed -ri "s/(memory_limit =) (.*$)/\1 -1/" /etc/php/7.2/cli/php.ini && \
  sed -ri 's/;(date.timezone =)/\1 Europe\/Amsterdam/' /etc/php/7.2/cli/php.ini && \
  sed -ri 's/listen\s*=\s*127.0.0.1:9000/listen = 9000/g' /etc/php/7.2/fpm/pool.d/www.conf && \
  sed -ri 's|;include_path = ".:/php/includes"|include_path = ".:/usr/share/php7.2"|g' /etc/php/7.2/fpm/php.ini && \
  mkdir -p /var/log/php7.2-fpm/ && \
  ln -sf /dev/stdout /var/log/php7.2-fpm.log && \
  ln -s /usr/sbin/php-fpm7.2 /usr/sbin/php-fpm7


# Clone nZEDb and set directory permissions
ENV NZEDB_VERSION "0.x"
RUN mkdir -p /var/www && \
  cd /var/www && \
  git clone https://github.com/nZEDb/nZEDb.git && \
  cd /var/www/nZEDb && \
  git checkout --quiet --force $NZEDB_VERSION && \
  composer install --prefer-dist && \
  chmod -R 777 /var/www/nZEDb/ && \
  # nuke all git repos' .git dir except for nzedb's .git dir to save space
  find . -name ".git" -type d | grep -v "\.\/\.git" | xargs rm -rf && \
  # nuke ~350MB of composer cache
  composer clear-cache

# install tmux
# RUN apt install -y tmux

RUN apt install -y \
libevent-dev build-essential git autotools-dev automake pkg-config ncurses-dev python && \
apt remove -y tmux && \
mkdir /temp/ && cd /temp/ && \
git clone https://github.com/tmux/tmux.git --branch 2.0 --single-branc && \
cd tmux && \
./autogen.sh && \
./configure && \
make -j4 && \
make install && \
make clean

# install par2
RUN apt-get install -y \
  par2 gzip

# Create dir for importing nzbs
RUN mkdir -p /var/www/nZEDb/resources/import

# Switch out php executable to instrument invocations
RUN mv /usr/bin/php /usr/bin/php.real
COPY php.proxy /usr/bin/php

# # Use pigz (parallel gzip) instead of gzip to speed up db backups
# RUN mv /bin/gzip /bin/gzip.real && \
#   ln -s /usr/bin/pigz /bin/gzip

# iconv has issues in musl which affects NFO conversion to include
# cool ascii chars. Remove the problematic parts - TRANSLIT and IGNORE
# See https://github.com/slydetector/simply-nzedb/issues/31
RUN sed -i "s|UTF-8//IGNORE//TRANSLIT|UTF-8|g" /var/www/nZEDb/nzedb/utility/Text.php

LABEL nzedb=$NZEDB_VERSION \
  maintainer=thewillonline \
  url=https://github.com/wimjan123/simply-nzedb

RUN mkdir -p /var/www/nZEDb/resources/tmp && chmod 777 /var/www/nZEDb/resources/tmp

RUN chmod 777 -R /var/lib/php/sessions


ENV TERM tmux
EXPOSE 8800
ADD s6 /etc/s6
CMD ["/bin/s6-svscan","/etc/s6"] && /usr/sbin/service apache2 start
WORKDIR /var/www/nZEDb/misc/update
