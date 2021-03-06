FROM wordpress:5-php7.2-apache

LABEL maintainer="support@computestacks.com"

ARG php_redis=4.2.0
ARG php_memcache=3.0.4
ARG wp_cli=2.1.0

RUN usermod -u 1001 www-data \
    && groupmod -g 1001 www-data

RUN set -ex; \
    \
    apt-get update; \
    apt-get -y upgrade; \
    apt-get install -y \
            zip \
            unzip \
            libpng-dev \
            libjpeg62-turbo-dev \
            mysql-client \
            libxml2-dev \
            libbz2-dev \
            libmemcached-dev \
            libfreetype6-dev \
            libcurl4-gnutls-dev \
            libc-client-dev \
            libkrb5-dev \
            libxslt-dev \
            vim \
            nano \
            zlib1g-dev \
            libicu-dev \
            g++ \
            wget \
            imagemagick \
            libmagickwand-dev \
    ; \
    rm -rf /var/lib/apt/lists/*;

RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install gd \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install imap \
    && docker-php-ext-configure intl \
    && docker-php-ext-install intl \
    && pecl install imagick-3.4.3 \
    && docker-php-ext-enable imagick \
    && docker-php-ext-install -j$(nproc) \
                                pdo_mysql \
                                mysqli \
                                soap \
                                mbstring \
                                opcache \
                                bcmath \
                                bz2 \
                                curl \
                                iconv \
                                ftp \
                                soap \
                                json \
                                posix \
                                session \
                                sockets \
                                xml \
                                xmlrpc \
                                xmlwriter \
                                xsl \
                                zip \
                                wddx

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=30'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN a2enmod rewrite expires deflate

COPY vendor/memcached-$php_memcache.tgz /tmp/memcached.tgz

RUN curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$php_redis.tar.gz \
    && tar -xzvf /tmp/redis.tar.gz -C /tmp \
    && mkdir -p /usr/src/php/ext \
    && mv /tmp/phpredis-$php_redis /usr/src/php/ext/redis \
    && rm -rf /tmp/redis.tar.gz \
    && docker-php-ext-install redis \
    && tar -xzvf /tmp/memcached.tgz \
    && mv memcached-$php_memcache /usr/src/php/ext/memcached \
    && rm /tmp/memcached.tgz \
    && docker-php-ext-install memcached \
    && wget https://downloads.wordpress.org/plugin/wp-mail-smtp.zip -O /tmp/smtptemp.zip \
    && cd /usr/src/wordpress/wp-content/plugins \
    && unzip /tmp/smtptemp.zip \
    && rm /tmp/smtptemp.zip \
    && chown www-data:www-data -R /usr/src/wordpress/wp-content/plugins \
    && rm /etc/apache2/sites-enabled/000-default.conf \
    && wget https://github.com/wp-cli/wp-cli/releases/download/v${wp_cli}/wp-cli-${wp_cli}.phar -O /usr/local/bin/wp \
    && chmod +x /usr/local/bin/wp

COPY 000-default.conf /etc/apache2/sites-enabled
COPY logging.conf /etc/apache2/conf-enabled
COPY lb_fix.sh /root
COPY docker-entrypoint.sh /usr/local/bin/

COPY cs_wp.ini /usr/local/etc/php/conf.d

RUN bash /root/lb_fix.sh
