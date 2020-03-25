#note: problem with ubuntu 18.04, curl needs libcurl4 but php7.3-curl needs libcurl13
#and 2 different version of libcurl can not co-exists

FROM ubuntu:16.04
MAINTAINER fahimu@datainterconnect.co.uk

RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y software-properties-common
RUN apt-get install -y sudo
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4F4EA0AAE5267A6C
RUN add-apt-repository "deb http://ppa.launchpad.net/ondrej/php/ubuntu xenial main"
#RUN add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"

RUN apt-get update

RUN apt-get install -y apt-file
RUN apt-file update
RUN apt-get install -y curl \
    vim \
    zip \
    python-pip \
    git

RUN pip install --upgrade awscli

RUN apt-get install -y --allow-unauthenticated php7.3 \
    php7.3-bcmath \
    php7.3-mbstring \
    php7.3-curl \
    php7.3-xml \
    php7.3-zip \
    php7.3-mysql \
    php7.3-gmp \
    libicu55 \
    php7.3-intl \
    php7.3-dev

#install apache
RUN update-alternatives --set php /usr/bin/php7.3
RUN apt-get install -y rsyslog \
    apache2

COPY optimus.di.conf /etc/apache2/sites-available/

RUN a2enmod rewrite
RUN a2enmod headers
RUN a2dissite 000-default
RUN a2ensite optimus.di.conf

#install xDebug
COPY xdebug-2.7.0RC1.tgz /tmp/xdebug-2.7.0RC1.tgz
RUN cd /tmp \
    && tar -xvzf xdebug-2.7.0RC1.tgz \
    && cd xdebug-2.7.0RC1 \
    && phpize \
    && ./configure \
    && make \
    && cp modules/xdebug.so /usr/lib/php/20180731

RUN echo "zend_extension = /usr/lib/php/20180731/xdebug.so" >> /etc/php/7.3/cli/php.ini
RUN echo "zend_extension = /usr/lib/php/20180731/xdebug.so" >> /etc/php/7.3/apache2/php.ini

RUN echo "[xdebug]" >> /etc/php/7.3/cli/php.ini \
    && echo "xdebug.idekey=PHPSTORM" >> /etc/php/7.3/cli/php.ini \
    && echo "xdebug.remote_enable=1" >> /etc/php/7.3/cli/php.ini \
    && echo "xdebug.remote_autostart=1" >> /etc/php/7.3/cli/php.ini \
    && echo "xdebug.remote_port=9000" >> /etc/php/7.3/cli/php.ini \
    && echo "xdebug.remote_host=172.17.0.1" >> /etc/php/7.3/cli/php.ini

RUN echo "[xdebug]" >> /etc/php/7.3/apache2/php.ini \
    && echo "xdebug.idekey=PHPSTORM" >> /etc/php/7.3/apache2/php.ini \
    && echo "xdebug.remote_enable=1" >> /etc/php/7.3/apache2/php.ini \
    && echo "xdebug.remote_port=9000" >> /etc/php/7.3/apache2/php.ini \
    && echo "xdebug.remote_connect_back=1" >> /etc/php/7.3/apache2/php.ini

#copy phpunit
COPY phpunit-6.5.phar /usr/local/bin/phpunit
RUN chmod +x /usr/local/bin/phpunit

#add user
RUN groupadd optimus
RUN useradd --create-home --shell /bin/bash  -g optimus optimus
RUN usermod -a -G sudo optimus
RUN echo '%optimus ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
RUN newgrp sudo

#install composer
RUN cd ~
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php
RUN php composer-setup.php --install-dir=/usr/local/bin --filename=composer

RUN echo "ServerName localhost" >> /etc/apache2/httpd.conf
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

EXPOSE 80 443 9000

CMD ["/usr/sbin/apachectl","-D","FOREGROUND"]
