FROM php:5.6-fpm-alpine

# Install packages and remove default server definition
RUN apk --no-cache add gnupg autoconf make g++ nginx supervisor zlib-dev libpng-dev icu-dev icu-libs git libzip-dev && \
    rm /etc/nginx/conf.d/default.conf

# Install PHP extensions
RUN apk add g++ gcc unixodbc-dev unixodbc freetds-dev freetds
RUN docker-php-ext-install bcmath gd exif pcntl intl zip calendar mssql && \ 
    docker-php-ext-configure mssql
# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer


# Configure nginx
COPY config/nginx.conf /etc/nginx/nginx.conf

# Configure PHP
RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
COPY config/php.ini /usr/local/etc/php/conf.d/custom.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Setup document root
RUN mkdir -p /var/www/html

# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody.nobody /var/www/html && \
  chown -R nobody.nobody /run && \
  chown -R nobody.nobody /var/lib/nginx && \
  chown -R nobody.nobody /var/log/nginx

# Switch to use a non-root user from here on
USER www-data

# Add application
WORKDIR /var/www/html

# Add a volume so that the external source code can be hooked
VOLUME [ "/var/www/html" ]

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
