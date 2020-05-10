FROM php:7.3-apache
MAINTAINER kazuki-yokomura <foxtail814@gmail.com>
LABEL Description="apache7.3 cakephp4" Version="1.0"

ENV DOCUMENT_ROOT="/srv/httpd/webroot"

# デフォルトでインストールされているモジュールの確認.
# Core
# ctype
# curl
# date
# dom
# fileinfo
# filter
# ftp
# hash
# iconv
# json
# libxml
# mbstring
# mysqlnd
# openssl
# pcre
# PDO
# pdo_sqlite
# Phar
# posix
# readline
# Reflection
# session
# SimpleXML
# sodium
# SPL
# sqlite3
# standard
# tokenizer
# xml
# xmlreader
# xmlwriter
# zlib

# linux パッケージインストール.
RUN apt-get update && apt-get install -y \
    # sh したときに必要?
    vim \
    # php-gd に必要
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    # pecl mcrypt に必要
    libmcrypt-dev \
    # cakePHP インストールに必要
    unzip \
    zip \
    libicu-dev \
    # デーモン起動に必要
    supervisor

# 足りない extension インストール.
RUN docker-php-ext-configure gd \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
    --with-freetype-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
        intl \
        pdo_mysql \
    && pecl install mcrypt \
        mysql

# mailhog 用 mhsendmail インストール
RUN curl -sSL https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 -o mhsendmail \
    && chmod +x mhsendmail \
    && mv mhsendmail /usr/local/bin/mhsendmail

COPY ./php.ini /usr/local/etc/php/php.ini 

# タイムゾーン設定.
RUN echo cp /usr/share/zoneinfo/Japan /etc/localtime

# ドキュメントルートの変更.(特に必須ではない)
RUN sed -ri -e "s!/var/www/html!${DOCUMENT_ROOT}!g" /etc/apache2/sites-available/*.conf \
 && sed -ri -e "s!/var/www/!${DOCUMENT_ROOT}!g" /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf \
 && sed -ri -e "172 s/None/All/g" /etc/apache2/apache2.conf

# mod_rewrite 有効化
RUN a2enmod rewrite

# 最新の composer インストール(Docker イメージに依存。).
# こんな感じで他のイメージからデータをとってこれることを、マルチステージビルドと言う.
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# ポートを開けておく.
EXPOSE 80
COPY ./supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# CMD は docker build するときではなく、docker run,すなわちコンテナを作る際に動きます。
# apache の起動や、mailcatcher の起動、Cakeのインストールは実態がないとできないため、CMDに書きます。
CMD ["/usr/bin/supervisord", "--configuration=/etc/supervisor/supervisord.conf"]
