FROM alpine:latest

RUN apk update \
  && apk add --no-cache nginx php7-fpm php7-mcrypt \
    php7-soap php7-openssl php7-gmp \
    php7-pdo_odbc php7-json php7-dom \
    php7-pdo php7-zip php7-mysqli \
    php7-apcu php7-pdo_pgsql \
    php7-bcmath php7-gd php7-odbc \
    php7-pdo_mysql \
    php7-gettext php7-xmlreader php7-xmlrpc \
    php7-bz2 php7-iconv php7-pdo_dblib php7-curl php7-ctype \
    libjpeg libjpeg-turbo libpng libpng-dev \
    supervisor

COPY nginx.conf /etc/nginx/nginx.conf
COPY configure.sh /configure.sh
COPY supervisord.conf /etc/supervisord.conf

VOLUME ["/var/lib/nginx/html/"]

EXPOSE 8080/tcp

RUN sh /configure.sh

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
