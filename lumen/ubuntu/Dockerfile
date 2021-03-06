FROM ubuntu:20.04

WORKDIR /var/www/html/

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV DEBIAN_FRONTEND="noninteractive"

# Setting up locale
RUN apt-get update \
    && apt-get install -y locales locales-all \
    && locale-gen en_US.UTF-8

# Essentials
RUN apt-get install -y zip \
    curl \
    unzip \
    nginx \
    sqlite3 \
    supervisor \
    software-properties-common

# Instaling PHP
RUN add-apt-repository -y ppa:ondrej/php && apt-get update -y

RUN apt-get install -y php8.0-common \
    php8.0-fpm \
    php8.0-pdo \
    php8.0-mbstring \
    php8.0-mysql \
    php8.0-sqlite3 \
    php8.0-xml \
    php8.0-curl \
    php8.0-zip \
    php8.0-cli \
    php8.0-readline \
    php8.0-gd \
    php8.0-imap \
    php8.0-xdebug \
    php-igbinary \
    php-mongodb \
    php-redis \
    php-json

# Installing composer
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm -rf composer-setup.php

# Configure php
COPY .docker/php.ini-production /etc/php/8.0/cli/php.ini

# Configure php-fpm
COPY .docker/php-fpm.conf /etc/php/8.0/fpm/php-fpm.conf
COPY .docker/fastcgi-php.conf /etc/nginx/fastcgi-php.conf

RUN mkdir -p /run/php/ \
    && touch /run/php/php8.0-fpm.pid \
    && touch /run/php/php8.0-fpm.sock

# Configure nginx
COPY .docker/default.conf /etc/nginx/conf.d/default.conf

RUN echo "daemon off;" >> /etc/nginx/nginx.conf \
    && mkdir -p /run/nginx/ && touch /run/nginx/nginx.pid

# Configure logs
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

# Configure supervisor
RUN mkdir -p /etc/supervisor.d/
COPY .docker/supervisord.ini /etc/supervisor/conf.d/supervisord.ini

# Cleaning up image
RUN apt-get remove -y --purge software-properties-common \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Building process
COPY . .
RUN composer install --no-dev

# Setting up permissions for Lumen storage
RUN chown -R www-data /var/www/html/storage

# Configure Lumen logs
RUN ln -sf /dev/stdout /var/www/html/storage/lumen.log

EXPOSE 80
CMD ["supervisord", "-c", "/etc/supervisor.d/supervisord.ini"]
