FROM php:7.4.12-fpm-buster

ENV USERNAME www-app

# --------------------------------------------------------------------------------------------------

# INSTALL SYSTEM DEPENDENCIES

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    # Required package
    software-properties-common \
    # System utils
    apt-utils \
    bash-completion \
    cron \
    curl \
    default-mysql-client \
    git \
    gzip \
    gnupg \
    procps \
    redis-tools \
    tar \
    unzip \
    vim \
    nano \
    wget \
    zip \
    # Required for extensions
    libcurl4-openssl-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libonig-dev \
    libpcre3-dev \
    libpng-dev \
    libssl-dev \
    libxml2-dev \
    libxslt1-dev \
    libzip-dev \
    zlib1g-dev \
    # Delete downloaded packages after installation
    && apt-get clean

# --------------------------------------------------------------------------------------------------

# INSTALL REQUIRED PHP EXTENSIONS

RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j"$(nproc)" \
    gd \
    bcmath \
    intl \
    mysqli \
    opcache \
    pdo_mysql \
    soap \
    sockets \
    xsl \
    zip
    # ctype -- Required but already loaded
    # curl -- Required but already loaded
    # dom -- Required but already loaded
    # hash -- Required but already loaded
    # iconv -- Required but already loaded
    # mbstring -- Required but already loaded
    # simplexml -- Required but already loaded
    # xml -- Required but already loaded

# --------------------------------------------------------------------------------------------------

# CREATE APPLICATION USER

RUN groupadd -g 1000 ${USERNAME} \
    && useradd -g 1000 -u 1000 -d /var/www -s /bin/bash ${USERNAME}

# Set new user as owner of NPM and Composer config folders
RUN mkdir /var/www/.config /var/www/.npm \
    && chown ${USERNAME}:${USERNAME} /var/www/.config /var/www/.npm

# --------------------------------------------------------------------------------------------------

# INSTALL COMPOSER

RUN curl -sS https://getcomposer.org/installer | php -- \
    --install-dir=/usr/local/bin --filename=composer --1

# Install Composer packages in parallel. See: https://github.com/hirak/prestissimo
RUN composer global require hirak/prestissimo

# --------------------------------------------------------------------------------------------------

# INSTALL NODE.JS AND NPM

RUN curl -sL https://deb.nodesource.com/setup_12.x | bash - && apt-get install -y nodejs

# --------------------------------------------------------------------------------------------------

# INSTALL MAILHOG SENDMAIL

RUN curl -sSLO https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 \
    && chmod +x mhsendmail_linux_amd64 \
    && mv mhsendmail_linux_amd64 /usr/local/bin/mhsendmail

# --------------------------------------------------------------------------------------------------

# INSTALL MAGERUN FOR MAGENTO 2
# See: https://github.com/netz98/n98-magerun2

RUN wget https://files.magerun.net/n98-magerun2.phar \
	&& chmod +x ./n98-magerun2.phar \
	&& mv ./n98-magerun2.phar /usr/local/bin/ \
    # Bash completion support
    && curl -o /etc/bash_completion.d/n98-magerun2.phar.bash \
    https://raw.githubusercontent.com/netz98/n98-magerun2/master/res/autocompletion/bash/n98-magerun2.phar.bash

# --------------------------------------------------------------------------------------------------

# INSTALL XDEBUG

RUN yes | pecl install xdebug-2.8.1 && docker-php-ext-enable xdebug

# Disable by default
RUN mv /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini.sample

# --------------------------------------------------------------------------------------------------

# ADD CUSTOM CONFIG FILES

COPY ./.docker/phpfpm/conf/php-fpm.conf /usr/local/etc/
COPY ./.docker/phpfpm/conf/php.ini /usr/local/etc/php/
COPY ./.docker/phpfpm/conf/xdebug.ini ./.docker/phpfpm/conf/www.conf /usr/local/etc/php/conf.d/
RUN echo "source /etc/bash_completion" >> /var/www/.bashrc

# --------------------------------------------------------------------------------------------------

# ADD CRON FILES

RUN printf '* *\t* * *\t${USERNAME}\t%s/usr/local/bin/php /var/www/html/update/cron.php\n' >> /etc/crontab \
    && printf '* *\t* * *\t${USERNAME}\t%s/usr/local/bin/php /var/www/html/bin/magento cron:run\n' >> /etc/crontab \
    && printf '* *\t* * *\t${USERNAME}\t%s/usr/local/bin/php /var/www/html/bin/magento setup:cron:run\n#\n' >> /etc/crontab

# --------------------------------------------------------------------------------------------------

# COPY APPLICATION FILES

COPY . /var/www/html

# --------------------------------------------------------------------------------------------------

# MIRROR VOLUMES ON NGINX AND PHP CONTAINERS

RUN mkdir -p /etc/nginx/html /var/www/html /sock \
    && chown -Rf ${USERNAME}:${USERNAME} /etc/nginx /var/www /usr/local/etc/php/conf.d /sock

# --------------------------------------------------------------------------------------------------

# FINAL TOUCHES

USER ${USERNAME}:${USERNAME}
VOLUME /var/www
WORKDIR /var/www/html
EXPOSE 9000
