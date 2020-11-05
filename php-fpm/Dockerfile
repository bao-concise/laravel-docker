#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

FROM php:7.4-fpm

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

#
#--------------------------------------------------------------------------
# Software's Installation
#--------------------------------------------------------------------------
#
# Installing tools and PHP extentions using "apt", "docker-php", "pecl",
#

# Install "curl", "libmemcached-dev", "libpq-dev", "libjpeg-dev",
#         "libpng-dev", "libfreetype6-dev", "libssl-dev", "libmcrypt-dev",
RUN set -eux; \
    apt-get update; \
    apt-get upgrade -y; \
    apt-get install -y --no-install-recommends \
            curl \
            libmemcached-dev \
            libz-dev \
            libpq-dev \
            libjpeg-dev \
            libpng-dev \
            libfreetype6-dev \
            libssl-dev \
            libmcrypt-dev \
            libonig-dev; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    # Install the PHP pdo_mysql extention
    docker-php-ext-install pdo_mysql; \
    # Install the PHP pdo_pgsql extention
    docker-php-ext-install pdo_pgsql; \
    # Install the PHP gd library
    docker-php-ext-configure gd \
            --prefix=/usr \
            --with-jpeg \
            --with-freetype; \
    docker-php-ext-install gd; \
    php -r 'var_dump(gd_info());'

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    apt-get install -yqq \
      apt-utils \
      #
      #--------------------------------------------------------------------------
      # Mandatory Software's Installation
      #--------------------------------------------------------------------------
      #
      # Mandatory Software's such as ("mcrypt", "pdo_mysql", "libssl-dev", ....)
      # are installed on the base image 'laradock/php-fpm' image. If you want
      # to add more Software's or remove existing one, you need to edit the
      # base image (https://github.com/Laradock/php-fpm).
      #
      # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
      libzip-dev zip unzip && \
      docker-php-ext-configure zip && \
      # Install the zip extension
      docker-php-ext-install zip && \
      php -m | grep -q 'zip'

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd


RUN pecl install xdebug && \
  docker-php-ext-enable xdebug

# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /usr/local/etc/php/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /usr/local/etc/php/conf.d/xdebug.ini

###########################################################################
# MySQL Client:
###########################################################################

USER root

RUN apt-get -y install default-mysql-client

###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN pecl install -o -f redis \
    && rm -rf /tmp/pear \
    && docker-php-ext-enable redis

###########################################################################
# ImageMagick:
###########################################################################

USER root

RUN apt-get install -y libmagickwand-dev imagemagick && \
    pecl install imagick && \
    docker-php-ext-enable imagick

###########################################################################
# Composer:
###########################################################################
USER root

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer


###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "PHP 7.4."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog


# Configure non-root user.
RUN groupmod -o -g 1000 www-data && \
    usermod -o -u 1000 -g www-data www-data

# Adding the faketime library to the preload file needs to be done last
# otherwise it will preload it for all commands that follow in this file
RUN echo "/usr/lib/x86_64-linux-gnu/faketime/libfaketime.so.1" > /etc/ld.so.preload


WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000