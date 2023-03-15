# How to Deploy a PHP Application with Nginx and MySQL Using Docker and Docker Compose for Developments

# Contents
 - [Prerequisites](#prerequisites) 
 - [Directory Structure](#directory-structure)
 - [Docker compose file](#docker-compose-file)
 - [Nginx default configuration file to run your PHP application](#nginx-default-configuration-file-to-run-your-php-application)
 - [Dockerfile for PHP-FPM](#dockerfile-for-php-fpm)
 - [Run the docker compose](#run-the-docker-compose)

## Prerequisites
- Docker runner in Linux or Docker-Desktop with WSL in Windows

## Directory Structure

```
├── docker-compose.yml
├── nginx
│   ├── default.conf
│   └── Dockerfile
├── php
│   └── Dockerfile
└── www
    └── html
        └── index.php
```

## Docker compose file

The file is shown below.

You can change the value of environment variables as per your need.

Remembering that if you change the root password environment variable MYSQL_ROOT_PASSWORD you will also have to change it in line 16 that tests if the database is up

<details><summary>docker-compose.yml</summary>
<p>

```yml showLineNumbers
version: '2.1'
services:
  db:
    image: mysql:5.7
    container_name: mysql-container
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=mydb
      - MYSQL_USER=myuser
      - MYSQL_PASSWORD=password
    ports:
      - '3306:3306'
    volumes:
      - './data:/var/lib/mysql'
    healthcheck:
      test: 'mysqladmin ping -h localhost -psecret'
      interval: 30s
      timeout: 30s
      retries: 3
  fpm:
    build: ./php/
    container_name: php-container
    expose:
      - 9000
    links:
      - db
    volumes:
      - "./www/html/:/var/www/html/"
    depends_on:
      db:
        condition: service_healthy
  web:
    build: ./nginx/
    container_name: nginx-container
    ports:
      - 8080:80
    links:
      - fpm
    volumes:
      - "./www/html/:/var/www/html/"
    depends_on:
      fpm:
        condition: service_started
```
</p>
</details>


## Nginx default configuration file to run your PHP application:

The nginx configuration file is in the path [nginx/default.conf](nginx/default.conf) and is also shown below.

<details><summary>nginx/default.conf</summary>
<p>

```nginx showLineNumbers
server {  

     listen 80 default_server;  
     root /var/www/html;  
     index index.html index.php;  

     charset utf-8;  

     location / {  
      try_files $uri $uri/ /index.php?$query_string;  
     }  

     location = /favicon.ico { access_log off; log_not_found off; }  
     location = /robots.txt { access_log off; log_not_found off; }  

     access_log off;  
     error_log /var/log/nginx/error.log error;  

     sendfile off;  

     client_max_body_size 100m;  

     location ~ .php$ {  
      fastcgi_split_path_info ^(.+.php)(/.+)$;  
      fastcgi_pass fpm:9000;  
      fastcgi_index index.php;  
      include fastcgi_params;  
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;  
      fastcgi_intercept_errors off;  
      fastcgi_buffer_size 16k;  
      fastcgi_buffers 4 16k;  
    }  

     location ~ /.ht {  
      deny all;  
     }  
    } 
```

</p>
</details>

## Dockerfile for PHP-FPM

The Dockerfile file is in the path [php/Dockerfile](php/Dockerfile) and is also shown below.

```Dockerfile showLineNumbers
FROM php:7.0-fpm
RUN docker-php-ext-install pdo_mysql 
```

If you need to add other php dependencies please change the dockerfile for your need.

## Run the docker compose

What everyone wanted, how to run containers

To start the services, run the command below to read the output and verify that all services have gone up correctly

```sh
$ docker composer up
```

Or to run the services in daemon mode run the command below

```sh  showLineNumbers
$ docker composer up -d
```