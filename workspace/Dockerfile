FROM php:7.1-fpm

# Start as root
USER root

###########################################################################
# adbert non-root user:
###########################################################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupadd -g ${PGID} adbert && \
    useradd -u ${PUID} -g adbert -m adbert -G docker_env && \
    usermod -p "*" adbert


###########################################################################
# Set Timezone
###########################################################################

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone



###########################################################################
# User Aliases
###########################################################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/adbert/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/adbert/aliases.sh && \
    chown adbert:adbert /home/adbert/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc

USER adbert

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc


###########################################################################
# Composer:
###########################################################################

USER root

# Add the composer.json
COPY ./composer.json /home/adbert/.composer/composer.json

# Make sure that ~/.composer belongs to adbert
RUN chown -R adbert:adbert /home/adbert/.composer

USER adbert

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    # run the install
    composer global install \
    ;fi

ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}

RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
    ;fi

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc



###########################################################################
# Crontab
###########################################################################

USER root

COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d


###########################################################################
# MongoDB:
###########################################################################

ARG INSTALL_MONGO=false
ARG PHP_VERSION=${PHP_VERSION}

RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    pecl -q install mongodb && \
    echo "extension=mongodb.so" >> /etc/php/${PHP_VERSION}/mods-available/mongodb.ini && \
    ln -s /etc/php/${PHP_VERSION}/mods-available/mongodb.ini /etc/php/${PHP_VERSION}/cli/conf.d/30-mongodb.ini \
    ;fi






WORKDIR /code/ats
ADD ./ /code/ats

RUN apt-get update \
    && apt-get install -y libxml2 libxml2-dev zip libzip-dev openssl libssh-dev libmcrypt-dev git wget \
    && docker-php-ext-install pdo pdo_mysql mbstring json zip mcrypt

RUN pecl install mongodb \
    && pecl install xdebug \
    && docker-php-ext-enable mongodb xdebug

RUN curl -sS https://getcomposer.org/installer | php -- --filename=composer --install-dir=/bin
ENV PATH /root/.composer/vendor/bin:$PATH

RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - \
    && apt-get install -y nodejs

RUN npm install -g pm2

EXPOSE 9000

CMD pm2 start pm2-listen/backend-queue.json && php-fpm
