FROM php:7.1-fpm

ARG APP_ENV=dev

RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libpq-dev \
	zlib1g-dev;

ADD https://git.archlinux.org/svntogit/packages.git/plain/trunk/freetype.patch?h=packages/php /tmp/freetype.patch
RUN docker-php-source extract; \
        cd /usr/src/php; \
        patch -p1 -i /tmp/freetype.patch; \
        rm /tmp/freetype.patch

RUN docker-php-ext-configure gd --with-gd --with-freetype-dir

RUN docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install -j$(nproc) zip \
    && docker-php-ext-install -j$(nproc) mbstring \
    && docker-php-ext-install -j$(nproc) exif \
    && docker-php-ext-enable exif \
    && docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
    && docker-php-ext-install -j$(nproc) mysqli pdo pdo_mysql pdo_pgsql pgsql

RUN if [ ${APP_ENV} = "dev" ]; then \
        php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer; \
    fi

WORKDIR /usr/share/nginx
