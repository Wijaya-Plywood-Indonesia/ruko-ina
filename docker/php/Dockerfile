FROM php:8.4-fpm

# Install dependencies sistem & ekstensi PHP yang dibutuhkan Laravel & Filament
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip \
    libicu-dev \
    libzip-dev \
    gnupg

# Install Node.js (Versi LTS terbaru via NodeSource) & npm
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - \
    && apt-get install -y nodejs

# Clear cache apt untuk memperkecil ukuran image
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install ekstensi PHP (termasuk intl dan zip; opcache untuk performa)
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd intl zip opcache

# [TAMBAHAN] Pengaturan PHP: batas upload (default PHP cuma 2M!)
RUN { \
    echo "upload_max_filesize=20M"; \
    echo "post_max_size=25M"; \
    echo "memory_limit=512M"; \
    echo "max_execution_time=120"; \
    echo "date.timezone=Asia/Jakarta"; \
    } > /usr/local/etc/php/conf.d/app.ini

# Get Composer (versi 2 dikunci agar build konsisten)
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

COPY composer.json composer.lock ./
RUN composer install --no-dev --optimize-autoloader --no-interaction --no-scripts

# --- Layer caching npm ---
COPY package.json package-lock.json ./
RUN npm ci

# Baru copy seluruh source code aplikasi (file yang di .dockerignore tidak ikut)
COPY . .

# Build asset frontend, lalu buang node_modules supaya image lebih kecil
RUN npm run build && rm -rf node_modules

# Jalankan composer scripts (package:discover, filament:upgrade) setelah semua file ada
RUN composer dump-autoload --optimize

# [TAMBAHAN] Buat symlink public/storage -> storage/app/public saat build,
# supaya ikut tersalin ke image nginx
RUN php artisan storage:link

# Beri hak tulis ke www-data (user PHP-FPM) untuk log, cache, & session
RUN mkdir -p storage/logs storage/framework/cache storage/framework/sessions storage/framework/views bootstrap/cache \
    && chown -R www-data:www-data storage bootstrap/cache

EXPOSE 9000
CMD ["php-fpm"]