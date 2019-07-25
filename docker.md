# Docker commands

## Installation

```bash
sudo yum update -y
sudo install docker
sudo service docker start
# add the ec2-user to the docker group
sudo usermod -a -G docker ec2-user
# restart/re log to pickup the new permissions

#test if you have permission, should not need sudo now
docker info 
```

## Create a docker file

```bash

touch docker
```

Edit the file, simply test below for amz linux AMI
```
FROM amazonlinux:2018.03
MAINTAINER You <you@you.com>

CMD ["echo", "hello from CMD"]

```

```bash
# create the image with a tag (0.1), add a space and . to indicate current dir
docker build -t testimage:0.1 .
# check if the image was created
docker images --filter reference=testimage

# and you should see something like:
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
testimage           0.1                 57f0f3a57abd        About a minute ago   167MB
```

Run the image to test it

```bash 
docker run testimage:0.1
# .. should output

hello from CMD

```

# clean up

```bash 
# remove the image now, use -f to force it
 docker rmi -f 57f0f3a57abd
```


# Creates a LAMP stack image using the official Amazon Linux image for Docker

Apache 2.4
PHP 7.0
MySQL 5.7
MongoDB 3.4
http and https (unsigned)
remote access for MySQL, MongoDB and Memcached

```
FROM amazonlinux:2018.03
MAINTAINER Your Name <you@you.com>

ADD create-user.sh /tmp/create-user.sh
ADD create-cert.sh /tmp/create-cert.sh
ADD install-php-mongo-library.sh /tmp/install-php-mongo-library.sh
ADD mongodb-org-3.4.repo /etc/yum.repos.d/mongodb-org-3.4.repo
ADD server-config.sh /tmp/server-config.sh
ADD start-servers.sh /usr/sbin/start-servers

RUN yum update -y && yum install -y \
sudo \
httpd24 \
mod24_ssl \
gcc \
memcached \
openssl-devel \
php70 \
php70-bcmath \
php70-cli \
php70-common \
php70-dba \
php70-dbg \
php70-devel \
php70-enchant \
php70-fpm \
php70-gd \
php70-gmp \
php70-imap \
php70-intl \
php70-json \
php70-ldap \
php70-mbstring \
php70-mcrypt \
php70-mysqlnd \
php70-odbc \
php70-opcache \
php70-pdo \
php70-pdo-dblib \
php70-pecl-igbinary \
php70-pecl-imagick \
php70-pecl-memcached \
php70-pecl-oauth \
php70-pecl-ssh2 \
php70-pecl-uuid \
php70-pecl-yaml \
php70-pgsql \
php70-process \
php70-pspell \
php70-recode \
php70-snmp \
php70-soap \
php70-tidy \
php70-xml \
php70-xmlrpc \
php70-zip \
php7-pear \
mysql57-server \
mongodb-org \
nano \
man \
&& yum clean all

EXPOSE 80
EXPOSE 443
EXPOSE 3306
EXPOSE 11211
EXPOSE 27017

RUN /bin/bash /tmp/create-user.sh && \
rm /tmp/create-user.sh && \
/bin/bash /tmp/create-cert.sh && \
rm /tmp/create-cert.sh && \
/bin/bash /tmp/install-php-mongo-library.sh && \
rm /tmp/install-php-mongo-library.sh && \
/bin/bash /tmp/server-config.sh && \
rm /tmp/server-config.sh

CMD /usr/bin/env bash start-servers;sleep infinity
```

