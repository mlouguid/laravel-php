<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

<p align="center">
<a href="https://github.com/laravel/framework/actions"><img src="https://github.com/laravel/framework/workflows/tests/badge.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>


--------------------------------------------------------------------------------------------------------------------------------------------

**Stack Laravel & Dockerfile PHP**


Les étapes pour démarrer une stack Laravel :

Dockerfile : laravelapp
```
FROM php:8.2-fpm

# Set working directory
WORKDIR /var/www/html

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl \
    libonig-dev \
    libzip-dev \
    libgd-dev \
    libxml2-dev # Add libxml2-dev

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl xml && \
    docker-php-ext-configure gd --with-external-gd && \
    docker-php-ext-install gd

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add user for laravel application
#RUN groupadd -g 1000 www-data && \
 #   useradd -u 1000 -ms /bin/bash -g www-data www-data

# Copy existing application directory contents
COPY . /var/www/html

# Copy existing application directory permissions
COPY --chown=www-data:www-data . /var/www/html

# Change current user to www
USER www-data

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
```

Commande line


$docker build -t laravelapp .

------------------------------------------------------------------------------------------------------------------------------------


- Portainer.yml



```
version: '3.8'

services:
  portainer:
    image: portainer/portainer-ce
    deploy:
      replicas: 1
      restart_policy:
        condition: any
      placement:
        constraints:
          - node.role == manager
    ports:
      - "9090:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer_data:/data
    networks:
      - portainer_net


networks:
  portainer_net:
    driver: overlay

volumes:
  portainer_data:
    driver: local
```

---------------------------------------------------------------------------------------------------

 laravel-app.yml


```
version: '3.8'
services:

  php:
    image: laravelapp:latest
    ports:
      - "9000:9000"
    working_dir: /var/www/html
    volumes:
      - ./:/var/www/html
      - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
      - ./php/php.ini:/usr/local/etc/php/conf.d/php.ini
    networks:
      - networkapp

  nginx:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./:/var/www/html
    networks:
      - networkapp

  mariadb:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: marouane
      MYSQL_USER: marouane
      MYSQL_PASSWORD: marouane
    ports:
      - "3306:3306"
    volumes:
      - ./mysql:/var/lib/mysql
      - ./mysql/my.cnf:/etc/mysql/my.cnf
    networks:
      - networkapp

networks:
  networkapp:
    driver: overlay

```
Commande line
$docker stack deploy -c laravel-app.yml laravel-app


![VyXpoPqnT0cyNQ06-image](https://github.com/mlouguid/laravel-php/assets/82709225/f3705201-e9da-42f1-8761-26e2302c609a)



You can use the numeric UID (User ID) and GID (Group ID) instead. To find out the UID and GID for the user www inside the container, you can run the following command:

docker exec -u 1000:1000 container-id id

This will show the UID and GID. Then, you can use these values to set the ownership:


```
sudo chown -R 1000:1000 /var/www/html
docker exec -it ID-Container Bash
composer install
composer update
```

deployer app-key laravel :

```
cp .env.example .env

php artisan key:generate
```

APP_URL=http://localhost

![jSSLSQed4ua2gqjn-image](https://github.com/mlouguid/laravel-php/assets/82709225/6e3916c3-a04b-4950-ac33-b7c09d6314c9)
