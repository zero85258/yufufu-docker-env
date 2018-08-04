FROM php:7.1-fpm

RUN apt-get update \
    && apt-get install -y libxml2 libxml2-dev zip libzip-dev openssl libssh-dev libmcrypt-dev git wget curl \
    && docker-php-ext-install pdo pdo_mysql mbstring json zip mcrypt


###########################################################################
# SOAP:
###########################################################################

ARG INSTALL_SOAP=false

RUN if [ ${INSTALL_SOAP} = true ]; then \
    # Install the soap extension
    # pecl install soap && \
    docker-php-ext-install soap \
;fi

###########################################################################
# MongoDB:
###########################################################################

ARG INSTALL_MONGO=false

RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    pecl install mongodb && \
    docker-php-ext-enable mongodb \
;fi

###########################################################################
# xDebug:
###########################################################################

ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Install the xdebug extension
    pecl install xdebug && \
    docker-php-ext-enable xdebug \
;fi

# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini


USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

RUN usermod -u 1000 www-data

WORKDIR /code

CMD ["php-fpm"]

EXPOSE 9000
