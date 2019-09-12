# Docker setup for LAMP stack development.
    Follow below steps or just git clone this repo.

### 1. Create directory structure & index file:

* `mkdir -p lamp_dock_project/www_host/html`
* `cd lamp_dock_project/`
* `echo "<?php phpinfo(); ?>" > www_host/html/index.php`
* `mkdir -p bin/webserver bin/mysql`

### 2. Add `docker-compose.yml` and `Dockerfile`s

* `vim docker-compose.yml`

```yaml
version: '3'
services:
    webserver:
        build: 
            context: ./bin/webserver
        container_name: 'php-webserver'
        restart: 'always'
        ports:
            - "80:80"
            - "443:443"
        volumes:
            - ./www_host:/var/www:z
        links:
            - 'mariadb'
    mariadb:
        build: ./bin/mysql
        container_name: 'php-mariadb'
        volumes:
            - mariadb:/var/lib/mysql
        restart: 'always'
        ports:
            - "3306:3306"
        environment:
            TZ: "Europe/Rome"
            MYSQL_ALLOW_EMPTY_PASSWORD: "no"
            MYSQL_ROOT_PASSWORD: "rootpwd"
            MYSQL_USER: 'testuser'
            MYSQL_PASSWORD: 'testpassword'
            MYSQL_DATABASE: 'testdb'

volumes:
    mariadb:

```

* `vim bin/mysql/Dockerfile`

```Dockerfile
FROM mariadb:10.1
```

* `vim bin/webserver/Dockerfile`

```Dockerfile
FROM php:7.1.20-apache

RUN apt-get -y update --fix-missing
# RUN apt-get upgrade -y

# Install useful tools
RUN apt-get -y install apt-utils vim wget dialog

# Install important libraries
RUN apt-get -y install --fix-missing apt-utils build-essential git curl libcurl3 libcurl3-dev zip

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install xdebug
# RUN pecl install xdebug-2.5.0
# RUN docker-php-ext-enable xdebug

# Other PHP7 Extensions

RUN apt-get -y install libmcrypt-dev
RUN docker-php-ext-install mcrypt

RUN apt-get -y install libsqlite3-dev libsqlite3-0 mysql-client
RUN docker-php-ext-install pdo_mysql 
RUN docker-php-ext-install pdo_sqlite
RUN docker-php-ext-install mysqli

RUN docker-php-ext-install curl
RUN docker-php-ext-install tokenizer
RUN docker-php-ext-install json

RUN apt-get -y install zlib1g-dev
RUN docker-php-ext-install zip

RUN apt-get -y install libicu-dev
RUN docker-php-ext-install -j$(nproc) intl

RUN docker-php-ext-install mbstring

RUN apt-get install -y libfreetype6-dev libjpeg62-turbo-dev libpng-dev
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ 
RUN docker-php-ext-install -j$(nproc) gd

# Enable apache modules
RUN a2enmod rewrite headers

```

### 3. Composer docker and set file permissions

* `docker-compose up -d`
* `sudo chmod -R 0775 www_host/`
* `sudo chown -R $USER:www-data www_host/`

### 4. Access from browser
* Open: `http://localhost/`

### 5. SSH into docker
* `docker ps`

* `docker exec -it <NAME> /bin/bash`

