---
title: docker-compose样例
categories: docker compose
tags:
  - docker compose
abbrlink: 56248
date: 2019-02-17 00:00:00
---

[![Join the chat at https://gitter.im/simpleyyt/jekyll-theme-next](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/jekyll-theme-next/lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

<!--more-->

### networks
- 创建新的网络

```
docker network create --driver=bridge --subnet=172.33.0.0/24 test_net
```
对比 docker-compose方式

```yaml
networks:
    test_net:
        driver: bridge
        ipam:
            driver: default
            config:
             - subnet: 172.33.0.0/24
```

### lnmp
- nginx/php
- [site.conf](https://github.com/SakuraGaara/docker-compose/blob/master/lnmp/conf.d/site.conf)

```yaml
version: '3'
services:
    nginx:
        image: nginx
        container_name: lnmp-nginx
        depends_on:
         - php
        ports:
         - "5008:80"
        networks:
         - "test_net"
        volumes:
         - ./conf.d/site.conf:/etc/nginx/conf.d/default.conf
         - ./www:/www
        links:
         - php:php
    php:
        image: php:5.6-fpm
        container_name: lnmp-php
        expose: 
         - 9000
        networks:
         - "test_net"
        volumes:
         - "./www:/www"
networks:
    test_net:
        driver: bridge
        ipam:
            driver: default
            config:
                - subnet: 172.32.0.0/24
```

### mysql 

```
docker run -p 3002:3306 --name mysql3302 -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
```
对比 docker-compose
```yaml
version: "3"

services:
    mysql1:
        image: mysql:5.6
        volumes:
         - ./data1:/var/lib/mysql
         - ./conf1:/etc/mysql
         - ./logs1:/logs
        container_name: mysql1
        ports:
         - 3303:3306
        environment:
            TZ: Asia/Shanghai
            MYSQL_ROOT_PASSWORD: 123456
        command:
            --character-set-server=utf8mb4
            --max_allowed_packet=32M
        networks:
         - mysql_net
    mysql2:
        image: mysql:5.6
        volumes:
         - ./data2:/var/lib/mysql
         - ./conf2:/etc/mysql
         - ./logs2:/logs
        container_name: mysql2
        ports:
         - 3304:3306
        environment:
            TZ: Asia/Shanghai
            MYSQL_ROOT_PASSWORD: 123456
        command:
            --character-set-server=utf8mb4
            --max_allowed_packet=32M
        networks:
         - mysql_net

networks:
    mysql_net:
        driver: bridge
        ipam:
            config:
                - subnet: 172.55.0.0/24
```

### build
- Dockerfile-app

```
FROM alpine:latest

RUN echo -e "http://mirrors.ustc.edu.cn/alpine/v3.7/main\n\
http://mirrors.ustc.edu.cn/alpine/v3.7/community" > /etc/apk/repositories

RUN apk --update add curl bash openjdk8-jre-base && \
      rm -rf /var/cache/apk/*


ENV JAVA_HOME /usr/lib/jvm/default-jvm
ENV PATH ${PATH}:${JAVA_HOME}/bin
```
- docker-compose-java.yml

```yaml
version: '3'
services:
    app:
        build:
            context: /data/alpine-java
            dockerfile: Dockerfile-java

```
- 运行 docker-compose -f docker-compose-java.yml build



