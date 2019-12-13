FROM php:7.3.6-fpm-alpine3.9

# Set working directory
WORKDIR /var/www

# Install dependencies
RUN apk update && apk add --no-cache openrc openssl bash mysql-client alpine-sdk autoconf vim php libzip-dev zip libzip-dev php7-pecl-imagick libpng-dev imagemagick imagemagick-libs imagemagick-dev

# memcached
RUN apk add memcached

ENV MEMCACHED_DEPS zlib-dev libmemcached-dev cyrus-sasl-dev
RUN apk add --no-cache --update libmemcached-libs zlib
RUN set -xe \
    && apk add --no-cache --update --virtual .phpize-deps $PHPIZE_DEPS \
    && apk add --no-cache --update --virtual .memcached-deps $MEMCACHED_DEPS \
    && pecl install memcached \
    && echo "extension=memcached.so" > /usr/local/etc/php/conf.d/20_memcached.ini \
    && rm -rf /usr/share/php7 \
    && rm -rf /tmp/* \
    && apk del .memcached-deps .phpize-deps

# Install extensions
RUN docker-php-ext-install sockets pdo pdo_mysql bcmath zip gd

# Imagick
RUN set -ex \
    && apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS imagemagick-dev libtool \
    && export CFLAGS="$PHP_CFLAGS" CPPFLAGS="$PHP_CPPFLAGS" LDFLAGS="$PHP_LDFLAGS" \
    && pecl install imagick-3.4.3 \
    && docker-php-ext-enable imagick \
    && apk add --no-cache --virtual .imagick-runtime-deps imagemagick \
    && apk del .phpize-deps

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN ln -s /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
    && adduser -D -g 'www' www \
    && chown -R www:www /var/www

# Copy existing application directory contents
COPY . /var/www

# Copy existing application directory permissions
COPY --chown=www:www . /var/www

# Change current user to www
USER www

EXPOSE 9000

# Making entrypoint
COPY ./entrypoint.sh /tmp/entrypoint.sh

ENTRYPOINT ["/tmp/entrypoint.sh"]
#CMD ["php-fpm"]