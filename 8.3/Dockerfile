FROM php:8.3-fpm

ARG MAGENTO_ROOT=/var/www/html

ENV PHP_MEMORY_LIMIT 2G
ENV PHP_VALIDATE_TIMESTAMPS 1
ENV DEBUG false
ENV MAGENTO_RUN_MODE production
ENV UPLOAD_MAX_FILESIZE 64M
ENV SENDMAIL_PATH /dev/null
ENV PHPRC ${MAGENTO_ROOT}/php.ini

ENV PHP_EXTENSIONS bcmath bz2 calendar exif gd gettext intl mysqli opcache pdo_mysql redis soap sockets sodium sysvmsg sysvsem sysvshm xsl zip pcntl

# Install dependencies
RUN apt-get update \
  && apt-get upgrade -y \
  && apt-get install -y --no-install-recommends \
  apt-utils \
  sendmail-bin \
  sendmail \
  sudo \
  iproute2 \
  gnupg2 \
  ca-certificates \
  lsb-release \
  software-properties-common \
  libbz2-dev \
  libjpeg62-turbo-dev \
  libpng-dev \
  libfreetype6-dev \
  libgmp-dev \
  libgpgme11-dev \
  libicu-dev \
  libldap2-dev \
  libpcre3-dev \
  libpspell-dev \
  libtidy-dev \
  libxslt1-dev \
  libyaml-dev \
  libzip-dev \
  zip \
  python3-yaml \
  && rm -rf /var/lib/apt/lists/*

# Configure the gd library
RUN docker-php-ext-configure \
  gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/
RUN docker-php-ext-configure \
  opcache --enable-opcache

# Install required PHP extensions
RUN docker-php-ext-install -j$(nproc) \
  bcmath \
  bz2 \
  calendar \
  exif \
  gd \
  gettext \
  gmp \
  intl \
  mysqli \
  opcache \
  pdo_mysql \
  pspell \
  shmop \
  soap \
  sockets \
  sysvmsg \
  sysvsem \
  sysvshm \
  tidy \
  xsl \
  zip \
  pcntl

RUN pecl install -o -f \
  gnupg \
  mailparse \
  oauth \
  pcov \
  redis \
  xmlrpc-1.0.0RC3 \
  yaml

COPY conf/etc/php-fpm.ini /usr/local/etc/php/conf.d/zz-magento.ini
COPY conf/etc/php-pcov.ini /usr/local/etc/php/conf.d/zz-pcov-settings.ini
COPY conf/etc/mail.ini /usr/local/etc/php/conf.d/zz-mail.ini
COPY conf/etc/php-fpm.conf /usr/local/etc/
COPY conf/etc/php-gnupg.ini /usr/local/etc/php/conf.d/gnupg.ini

RUN groupadd -g 1000 www && useradd -g 1000 -u 1000 -d ${MAGENTO_ROOT} -s /bin/bash www

COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN ["chmod", "+x", "/docker-entrypoint.sh"]

RUN mkdir -p ${MAGENTO_ROOT}

RUN chown -R www:www /usr/local /var/www /var/log /usr/local/etc/php/conf.d ${MAGENTO_ROOT}

ENTRYPOINT ["/docker-entrypoint.sh"]

WORKDIR ${MAGENTO_ROOT}

USER root

CMD ["php-fpm", "-R"]
