FROM php:7.2.15-fpm-alpine3.8
LABEL maintainer Tom Keur <mail@tomkeur.net>

# persistent / runtime depsfmusl
ENV PHPIZE_DEPS \
    autoconf \
    cmake \
    file \
    g++ \
    gcc \
    libc-dev \
    make \
    git \
    pkgconf \
    re2c \
    gettext-dev \
    pcre-dev

RUN apk add --no-cache --virtual .persistent-deps \
    # for intl extension
    icu-dev \
    # for soap
    libxml2-dev \
    # for yaml
    yaml-dev \
    # Gettext
    musl-dev \
    libintl \
    gettext

RUN set -xe \
    && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
    && docker-php-ext-configure bcmath --enable-bcmath \
    && docker-php-ext-configure intl --enable-intl \
    && docker-php-ext-configure pcntl --enable-pcntl \
    && docker-php-ext-configure pdo_mysql --with-pdo-mysql \
    && docker-php-ext-configure mbstring --enable-mbstring \
    && docker-php-ext-configure soap --enable-soap \
    && docker-php-ext-install -j "$(nproc)" \
        bcmath \
        intl \
        opcache \
        shmop \
        pcntl \
        pdo_mysql \
        mbstring \
        soap \
        sysvmsg \
        sysvshm \
        sysvsem \
        exif \
        zip \
        gettext \
        sockets \
    && pecl install \
       APCu \
       ds \
    && docker-php-ext-enable \
       apcu \
       ds \
    && apk del .build-deps \
    && rm -rf /tmp/*

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

CMD ["php-fpm"]
