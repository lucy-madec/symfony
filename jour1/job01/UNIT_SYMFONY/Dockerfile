FROM php:8.2-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    libzip-dev \
    libicu-dev

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql \
    && docker-php-ext-install intl \
    && docker-php-ext-install opcache \
    && docker-php-ext-install zip \
    && docker-php-ext-install exif \
    && docker-php-ext-install pcntl

# Install APCu and APC backward compatibility
RUN pecl install apcu \
    && docker-php-ext-enable apcu \
    && pecl clear-cache

# Configure PHP
COPY php.ini /usr/local/etc/php/conf.d/app.ini

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set working directory
WORKDIR /var/www/html/app

# Copy existing application directory
COPY ./app /var/www/html/app

RUN chown -R www-data:www-data /var/www/html/app \
    && chmod -R 775 /var/www/html/app/var
