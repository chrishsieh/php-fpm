FROM php:fpm-alpine

LABEL \
	name="PHP-FPM Image" \
	image="chrishsieh/php-fpm:alpine" \
    license="MIT"

ENV BUILD_DEPS="cmake icu-dev freetype-dev libjpeg-turbo-dev libpng-dev libzip-dev imagemagick-dev gettext-dev" \
    RUN_DEPS="sudo dumb-init libarchive-tools icu gettext jq zip git wget \
            freetype libjpeg-turbo libpng libzip mysql-client imagemagick \
            socat supervisor ca-certificates bash ncurses"

RUN set -x \
# ---- Installing Build dependencies ----
    && apk --update add --no-cache --virtual .build-deps ${PHPIZE_DEPS} ${BUILD_DEPS} \
# ---- Update PHP pecl ----
    && pecl channel-update pecl.php.net \
# ---- Installing PHP Extension: exif ----
	&& /usr/local/bin/docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) exif \
	&& (rm -rf /usr/local/lib/php/test/exif || true) \
    && (rm -rf /usr/local/lib/php/doc/exif || true) \
# ---- Installing PHP Extension: gd ----
    && /usr/local/bin/docker-php-ext-configure gd --with-gd --with-png-dir=/usr --with-zlib-dir=/usr --with-jpeg-dir=/usr --with-freetype-dir=/usr --enable-gd-jis-conv \
    && NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
    && /usr/local/bin/docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) gd \
    && (rm -rf /usr/local/lib/php/test/gd || true) \
    && (rm -rf /usr/local/lib/php/doc/gd || true) \
# ---- Installing PHP Extension: gettext ----
	&& /usr/local/bin/docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) gettext \
	&& (rm -rf /usr/local/lib/php/test/gettext || true) \
    && (rm -rf /usr/local/lib/php/doc/gettext || true) \
# ---- Installing PHP Extension: imagick ----
	&& pecl install imagick \
	&& docker-php-ext-enable imagick \
	&& (rm -rf /usr/local/lib/php/test/imagick || true) \
	&& (rm -rf /usr/local/lib/php/doc/imagick || true) \
# ---- Installing PHP Extension: intl ----
	&& /usr/local/bin/docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) intl \
	&& (rm -rf /usr/local/lib/php/test/intl || true) \
    && (rm -rf /usr/local/lib/php/doc/intl || true) \
# ---- Installing PHP Extension: mysqli ----
	&& /usr/local/bin/docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) mysqli \
	&& (rm -rf /usr/local/lib/php/test/mysqli || true) \
    && (rm -rf /usr/local/lib/php/doc/mysqli || true) \
# ---- Installing PHP Extension: pdo_mysql ----
	&& /usr/local/bin/docker-php-ext-configure pdo_mysql --with-zlib-dir=/usr \
	&& /usr/local/bin/docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) pdo_mysql \
	&& (rm -rf /usr/local/lib/php/test/pdo_mysql || true) \
    && (rm -rf /usr/local/lib/php/doc/pdo_mysql || true) \
# ---- Installing PHP Extension: xdebug ----
    && pecl install xdebug \
    && docker-php-ext-enable xdebug \
    && (rm -rf /usr/local/lib/php/test/xdebug || true) \
    && (rm -rf /usr/local/lib/php/doc/xdebug || true) \
# ---- Installing PHP Extension: zip ----
	&& /usr/local/bin/docker-php-ext-configure zip --with-zlib-dir=/usr --with-pcre-dir=/usr --with-libzip \
	&& /usr/local/bin/docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) zip \
	&& (rm -rf /usr/local/lib/php/test/zip || true) \
	&& (rm -rf /usr/local/lib/php/doc/zip || true) \
# ---- Removing Build dependencies ----
    && apk del .build-deps \
# ---- Installing Running dependencies ----
    && apk --update add ${RUN_DEPS} \
    && update-ca-certificates \
# ---- Installing Running env ----
    && rm -rf /etc/supervisor* \
    && mkdir -p /etc/supervisor/conf.d \
    && mkdir -p /var/log/supervisor \
    && mkdir -p /run.d \
    && cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini

WORKDIR /var/www
COPY supervisord.conf /etc/supervisor/supervisord.conf
COPY entrypoint.sh /entrypoint.sh
COPY func.d/*.sh /func.d/
RUN chmod +x /entrypoint.sh && chmod +x /func.d/*.sh

ENTRYPOINT ["dumb-init", "/entrypoint.sh"]