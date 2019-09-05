---
title:  利用Docker容器部署thinkphp应用
tags: docker
categories: php
---

最近要部署一个thinkphp的项目，由于要和其他系统并存，为了隔离环境，采用Docker容器方式进行部署。

Dockerfile如下：
```
FROM php:apache
COPY src/ /var/www/html/

ENV TZ Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime
RUN echo ${TZ} > /etc/timezone

ENV APACHE_DOCUMENT_ROOT /var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

RUN apt-get update && apt-get install -y \
    libjpeg62-turbo-dev \
    libpng-dev \
    libfreetype6-dev
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-configure gd \
    --with-freetype-dir \
    --with-jpeg-dir
RUN docker-php-ext-install gd

RUN a2enmod rewrite
RUN chmod -R 0755 /var/www/html
RUN chown -R www-data:www-data /var/www/html
```
# 要点
* php容器的快捷命令: `a2enmod` `docker-php-ext-install` `docker-php-ext-configure`等
* docker-php-ext-install安装扩展的时候注意顺序，尽量在configure本模块之后
