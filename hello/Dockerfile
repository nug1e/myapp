#
# nginx + pagespeed + substitutions + vhosts VTS + more headers + geoip 
# with php-fpm with lots of extensions base image build on Alpine Linux
# 
# credits: 
# - https://github.com/lagun4ik/docker-nginx-pagespeed
#


FROM php:7.0-fpm-alpine

MAINTAINER Nugroho Andi Hastanto <slemania@gmail.com>

#
# --------------------------------------------------------------------------
# Install PHP-FPM
# --------------------------------------------------------------------------
#
RUN apk update && \
    apk add \
        gcc \
        g++ \
        git \
        re2c \
        curl \
        make \
        libtool \
        libmagic \
        autoconf \
        icu-dev \
        zlib-dev \
        file-dev \
        freetds-dev \
        freetype-dev \
        gmp-dev \
        curl-dev \
        krb5-dev \
        imap-dev \
        bzip2-dev \
        erlang-dev \
        sqlite-dev \
        ghostscript \
        libexif-dev \
        libxslt-dev \
        poppler-utils \
        openssl-dev \
        openldap-dev \
        enchant-dev \
        unixodbc-dev \
        cyrus-sasl-dev \
        postgresql-dev \
        imagemagick-dev \
        libpq \
        libltdl \
        libjpeg \
        libpng-dev \
        libxpm-dev \
        libvpx-dev \
        libxml2-dev \
        libwebp-dev \
        libssh2-dev \
        libmcrypt-dev \
        libmemcached-dev \
        libjpeg-turbo-dev

RUN docker-php-ext-configure hash && \
    docker-php-ext-configure gd \
        --enable-gd-native-ttf \
        --with-vpx-dir=/usr/lib \
        --with-xpm-dir=/usr/lib \
        --with-webp-dir=/usr/lib \
        --with-jpeg-dir=/usr/lib \
        --with-freetype-dir=/usr/include/freetype2 && \
    docker-php-ext-install gd && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl && \
    docker-php-ext-configure ldap && \
    docker-php-ext-install ldap

# Install bcmath, mbstring and zip extensions
RUN docker-php-ext-install bcmath mbstring zip wddx shmop sockets

# Enable Exif PHP extentions requirements
RUN docker-php-ext-install exif gettext

# curl extension
RUN docker-php-ext-install curl

# ssh2 module
RUN pecl install ssh2-1.0 && \
    docker-php-ext-enable ssh2

RUN export CFLAGS="-I/usr/src/php" && \
    docker-php-ext-install xmlreader xmlwriter pdo_sqlite

RUN docker-php-ext-install \
    dom \
    bz2 \
    json \
    iconv \
    pcntl \
    phar \
    posix \
    simplexml \
    soap \
    xml \
    xmlrpc \
    xsl \
    calendar \
    session \
    ctype \
    fileinfo \
    ftp \
    sysvmsg \
    sysvsem \
    sysvshm    



#
# --------------------------------------------------------------------------
# Compile Nginx
# --------------------------------------------------------------------------
#

ARG NGINX_VERSION=1.13.2
ARG LIBPNG_VERSION=1.2.57
ARG MAKE_J=4

ENV NGINX_VERSION=${NGINX_VERSION} \
    LIBPNG_VERSION=${LIBPNG_VERSION} \
    MAKE_J=${MAKE_J}

RUN apk upgrade --no-cache --update && \
    apk add --no-cache --update \
        bash \
        ca-certificates \
        libuuid \
        apr \
        apr-util \
        libjpeg-turbo \
        icu \
        icu-libs \
        openssl \
        pcre \
        zlib

RUN set -x && \
    apk --no-cache add -t .build-deps \
        apache2-dev \
        apr-dev \
        apr-util-dev \
        build-base \
        curl \
        git \
        icu-dev \
        geoip-dev \
        libjpeg-turbo-dev \
        linux-headers \
        gperf \
        openssl-dev \
        pcre-dev \
        python \
        zlib-dev

# Build libpng
RUN cd /tmp && \
    curl -L http://prdownloads.sourceforge.net/libpng/libpng-${LIBPNG_VERSION}.tar.gz | tar -zx && \
    cd /tmp/libpng-${LIBPNG_VERSION} && \
    ./configure --build=$CBUILD --host=$CHOST --prefix=/usr --enable-shared --with-libpng-compat && \
    make -j${MAKE_J} install V=0 

# Build in additional Nginx modules
RUN cd /tmp && \
    git clone https://github.com/vozlt/nginx-module-vts.git && \
    git clone https://github.com/openresty/headers-more-nginx-module.git && \
    git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module.git

# Clean-up
RUN cd && \
    # Forward request and error logs to docker log collector
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \

#
# --------------------------------------------------------------------------
# PHP-FPM Final Touches
# --------------------------------------------------------------------------
#

# composer
RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer

# Set default work directory
ADD ./php-fpm/config/*.ini  /usr/local/etc/php/conf.d/
ADD ./php-fpm/config/*.conf /usr/local/etc/php-fpm.d/

# create PHP user
RUN addgroup php-fpm -g 1000 && \
    adduser -h /usr/share/php -u 1000 -G php-fpm -D -s /bin/false php-fpm

# remove access log for health check
RUN sed -i '/^access.log/ d' /usr/local/etc/php-fpm.d/docker.conf



#
# --------------------------------------------------------------------------
# Nginx Final Touches
# --------------------------------------------------------------------------
#

# Inject Nginx configuration files
ADD ./nginx/conf.d /etc/nginx/conf.d
ADD ./nginx/include /etc/nginx/include
ADD ./nginx/nginx.conf /etc/nginx/nginx.conf
ADD ./nginx/fastcgi_params /etc/nginx/fastcgi_params
ADD ./docker-entrypoint.sh /usr/local/bin/

RUN echo "<?php phpinfo();" > /etc/nginx/html/info.php
RUN chmod +x /usr/local/bin/*


# Removing build dependencies, clean temporary files
RUN apk del .build-deps && \
    docker-php-source delete && \
    rm -rf /var/cache/apk/* /var/tmp/* /tmp/* 



#
# --------------------------------------------------------------------------
# Touchdown!
# --------------------------------------------------------------------------
#

EXPOSE 80
EXPOSE 8080

# check if response header returns 200 code OR die
HEALTHCHECK --interval=5s --timeout=5s CMD [ "200" = "$(curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1/healthcheck)" ] || exit 1

CMD ["nginx", "-g", "daemon off;"]
ENTRYPOINT ["docker-entrypoint.sh"]