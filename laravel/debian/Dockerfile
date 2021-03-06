FROM debian:buster-slim

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
RUN apt-get update -y && apt-get install -y lsb-release \
    apt-transport-https \
    ca-certificates \
    zip \
    wget \
    curl \
    unzip \
    nginx \
    sqlite3 \
    supervisor

# Instaling PHP
RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
RUN echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list

RUN apt-get update -y && apt-get install -y php8.0-common \
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
    php-igbinary \
    php-mongodb \
    php-redis \
    php-json

# Installing composer
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm -rf composer-setup.php

# Configure php-fpm
COPY .docker/php-fpm.conf /etc/php/8.0/fpm/php-fpm.conf
COPY .docker/php.ini-production /etc/php/8.0/fpm/php.ini
RUN mkdir -p /run/php/ && touch /run/php/php8.0-fpm.pid

# Configure nginx
COPY .docker/nginx.conf /etc/nginx/
COPY .docker/nginx-laravel.conf /etc/nginx/modules/

RUN mkdir -p /run/nginx/ && touch /run/nginx/nginx.pid

# Configure logs
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

# Configure supervisor
RUN mkdir -p /etc/supervisor.d/
COPY .docker/supervisord.ini /etc/supervisor.d/supervisord.ini

# Cleaning up image
RUN apt-get remove -y --purge software-properties-common \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Building process
COPY . .
RUN composer install --no-dev
RUN chown -R www-data:www-data /var/www/html/storage

EXPOSE 80
CMD ["supervisord", "-c", "/etc/supervisor.d/supervisord.ini"]
