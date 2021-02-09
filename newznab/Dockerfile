FROM ubuntu:18.04
MAINTAINER thewillonline

ENV TZ 'Europe/Amsterdam'

RUN apt update -y && \
    apt upgrade -y && \
    apt install -y \
    ssh screen apache2 php php-fpm && \
    apt install - y \
    php-pear php-gd php-mysql php-redis php-curl php-json php-mbstring wget lame mediainfo subversion ffmpeg redis



#Install Rar & Unrar 6
RUN mkdir /rarinstall && \
  cd /rarinstall && \
  wget https://www.rarlab.com/rar/rarlinux-x64-6.0.0.tar.gz && \
  tar xfv rarlinux* && \
  cd rar && \
  cp unrar /usr/bin/ && cp rar /usr/bin && \
  rm /rarinstall -r
