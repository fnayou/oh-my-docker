# See https://github.com/docker-library/php/blob/4677ca134fe48d20c820a19becb99198824d78e3/7.0/fpm/Dockerfile
FROM php:7.2-fpm

LABEL maintainer="fnayou.aymen@gmail.com"

# Global dependencies
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    # dependencies for PHP extensions
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libicu-dev \
    libpq-dev \
    libxml2-dev \
    libjpeg-dev \
    && pecl install xdebug

# PHP extensions
RUN docker-php-ext-install \
    opcache \
    iconv \
    mbstring \
    intl \
    pdo \
    pdo_mysql \
    pdo_pgsql \
    gd

# MCrypt
RUN apt-get update -y && \
    apt-get install -y libmcrypt-dev && \
    pecl install mcrypt-1.0.1 && \
    docker-php-ext-enable mcrypt

# GD
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/

# xdebug
RUN docker-php-ext-enable xdebug
RUN { \
        echo "error_reporting = E_ALL"; \
        echo "display_startup_errors = On"; \
        echo "display_errors = On"; \
        echo "xdebug.remote_enable=1"; \
        echo "xdebug.remote_connect_back=1"; \
        echo "xdebug.idekey=\"PHPSTORM\""; \
        echo "xdebug.remote_port=9001"; \
    } >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

# opcache
RUN { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=60'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini
RUN echo "realpath_cache_size = 4096k; realpath_cache_ttl = 7200;" > /usr/local/etc/php/conf.d/php.ini

# composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && composer --version

# code sniffer
RUN pear install PHP_CodeSniffer
RUN /usr/local/bin/phpcs --config-set installed_paths /var/www/html/vendor/escapestudios/symfony2-coding-standard

# timezone
RUN rm /etc/localtime && ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime && "date"

RUN usermod -u 1000 www-data

CMD ["php-fpm"]
