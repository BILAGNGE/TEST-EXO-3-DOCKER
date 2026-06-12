# ── Image de base : PHP 8.3 FPM ──────────────────────────────────────────────
FROM php:8.3-fpm

# ── Extensions système requises par WordPress ─────────────────────────────────
RUN apt-get update && apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libzip-dev \
    libicu-dev \
    libonig-dev \
    unzip \
    curl \
    && docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
        --with-webp \
    && docker-php-ext-install -j$(nproc) \
        gd \
        mysqli \
        pdo \
        pdo_mysql \
        zip \
        intl \
        opcache \
        mbstring \
        exif \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ── Configuration PHP recommandée WordPress ───────────────────────────────────
RUN { \
    echo 'upload_max_filesize = 64M'; \
    echo 'post_max_size = 64M'; \
    echo 'memory_limit = 256M'; \
    echo 'max_execution_time = 300'; \
    echo 'max_input_time = 300'; \
    echo 'opcache.enable=1'; \
    echo 'opcache.memory_consumption=128'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=2'; \
} > /usr/local/etc/php/conf.d/wordpress.ini

# ── Téléchargement de la dernière version de WordPress ────────────────────────
RUN curl -o /tmp/wordpress.tar.gz https://wordpress.org/latest.tar.gz \
    && tar -xzf /tmp/wordpress.tar.gz -C /tmp \
    && mkdir -p /var/www/html \
    && cp -r /tmp/wordpress/. /var/www/html/ \
    && rm -rf /tmp/wordpress.tar.gz /tmp/wordpress \
    && chown -R www-data:www-data /var/www/html \
    && chmod -R 755 /var/www/html

WORKDIR /var/www/html
