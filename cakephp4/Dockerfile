FROM php:7.3-apache
MAINTAINER kazuki-yokomura <foxtail814@gmail.com>
LABEL Description="apache7.3 cakephp4" Version="1.0"

ENV DOCUMENT_ROOT="/srv/httpd/webroot" \
    USER="dockuser" \
    PASSWD="dockuser" \
    GROUP="dockusers"

# RUN echo "========== php インストールされたモジュールの確認. =========="
# RUN php -m
# デフォルトでインストールされているエクステンションモジュール
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

# RUN echo "========== linux パッケージインストール. =========="
RUN apt-get update && apt-get install -y \
    # ssh 接続に必要
    openssh-server \
    sudo \
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
    # http と ssh 接続の両立に必要
    supervisor

# RUN echo "========== 足りない extension インストール. =========="
RUN docker-php-ext-configure gd \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
    --with-freetype-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
        intl \
        pdo_mysql \
    && pecl install mcrypt \
        mysql

# RUN echo "========== タイムゾーン設定 =========="
RUN echo cp /usr/share/zoneinfo/Japan /etc/localtime

# RUN echo "========== ドキュメントルートの変更 =========="
RUN sed -ri -e "s!/var/www/html!${DOCUMENT_ROOT}!g" /etc/apache2/sites-available/*.conf
RUN sed -ri -e "s!/var/www/!${DOCUMENT_ROOT}!g" /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# RUN echo "========== 最新の composer インストール(Docker イメージに依存) =========="
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# RUN echo "========== ssh 接続をできるようにする. =========="
RUN mkdir /var/run/sshd

RUN useradd $USER \
    && echo "${USER}:${PASSWD}" | chpasswd \
    && mkdir /home/${USER}

RUN sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

RUN groupadd $GROUP

RUN echo "%${GROUP} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/${GROUP}
RUN usermod -aG $GROUP $USER

ADD ./files/supervisord.conf /etc/supervisor/supervisord.conf
ADD ./files/info.php /srv/httpd/webroot

# RUN echo "========== ポートを開けておく. =========="
EXPOSE 80 22
CMD ["/usr/bin/supervisord", "--configuration=/etc/supervisor/supervisord.conf"]

# TODO ssh したときの bash フォーマット
# TODO Database driver Cake\Database\Driver\Mysql cannot be used due to a missing PHP extension or unmet dependency
# TODO ssh したときの bash 左右したときのあれ
