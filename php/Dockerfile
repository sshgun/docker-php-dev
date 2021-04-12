FROM php:7.4-fpm

ARG USER
ARG USER_ID
ARG USER_GID


EXPOSE 3030

# Clean up
RUN apt-get remove -y git && apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# ADD resolv.conf /etc/resolv.conf
ADD cacert.pem /usr/local/share/ca-certificates/cacert.pem
RUN chmod 644 /usr/local/share/ca-certificates/cacert.pem && update-ca-certificates

# RUN apt-get -o Acquire::Check-Valid-Until=false update
# RUN apt-get install -y apt-transport-https
RUN apt-get -y update
RUN apt-get -y install gcc make autoconf libc-dev pkg-config libzip-dev

RUN apt-get install -y --no-install-recommends \
	git \
	libmemcached-dev \
	libz-dev \
	libpq-dev \
	libssl-dev libssl-doc libsasl2-dev \
	libmcrypt-dev \
	libxml2-dev \
	zlib1g-dev libicu-dev g++ \
	libldap2-dev libbz2-dev \
	curl libcurl4-openssl-dev \
	libenchant-dev libgmp-dev firebird-dev libib-util \
	re2c libpng++-dev \
	libwebp-dev libjpeg-dev libjpeg62-turbo-dev libpng-dev libxpm-dev libvpx-dev libfreetype6-dev \
	libmagick++-dev \
	libmagickwand-dev \
	zlib1g-dev libgd-dev \
	libtidy-dev libxslt1-dev libmagic-dev libexif-dev file \
	sqlite3 libsqlite3-dev libxslt-dev \
	libmhash2 libmhash-dev libc-client-dev libkrb5-dev libssh2-1-dev \
	unzip libpcre3 libpcre3-dev \
	poppler-utils ghostscript libmagickwand-6.q16-dev libsnmp-dev libedit-dev libreadline6-dev libsodium-dev \
	freetds-bin freetds-dev freetds-common libct4 libsybdb5 tdsodbc libreadline-dev librecode-dev libpspell-dev libonig-dev \
	supervisor

# fix for docker-php-ext-install pdo_dblib
# https://stackoverflow.com/questions/43617752/docker-php-and-freetds-cannot-find-freetds-in-know-installation-directories
RUN ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/

# RUN docker-php-ext-configure hash --with-mhash && \
# 	docker-php-ext-install hash
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl && \
	docker-php-ext-install imap iconv

RUN docker-php-ext-install bcmath bz2 calendar ctype curl dba dom enchant
RUN docker-php-ext-install fileinfo exif ftp gettext gmp
RUN docker-php-ext-install intl json ldap mbstring mysqli
RUN docker-php-ext-install opcache pcntl pspell
RUN docker-php-ext-install pdo pdo_dblib pdo_mysql pdo_pgsql pdo_sqlite pgsql phar posix
RUN docker-php-ext-install readline
RUN docker-php-ext-install session shmop simplexml soap sockets sodium
RUN docker-php-ext-install sysvmsg sysvsem sysvshm

# RUN docker-php-ext-install snmp

# fix for docker-php-ext-install xmlreader
# https://github.com/docker-library/php/issues/373
RUN export CFLAGS="-I/usr/src/php" && docker-php-ext-install xmlreader xmlwriter xml xmlrpc xsl

RUN docker-php-ext-install tidy tokenizer zend_test zip

# already build in... what they say...
# RUN docker-php-ext-install filter reflection spl standard
# RUN docker-php-ext-install pdo_firebird pdo_oci

# install pecl extension
RUN pecl install ds && \
	pecl install imagick && \
	pecl install igbinary && \
	pecl install memcached && \
	pecl install redis-5.1.0 && \
	pecl install mcrypt-1.0.3 && \
	docker-php-ext-enable ds imagick igbinary redis memcached

# https://serverpilot.io/docs/how-to-install-the-php-ssh2-extension
# 	pecl install ssh2-1.1.2 && \
# docker-php-ext-enable ssh2

# install pecl extension
# RUN pecl install mongodb && docker-php-ext-enable mongodb

# install xdebug
RUN pecl install xdebug && docker-php-ext-enable xdebug

# RUN yes "" | pecl install msgpack && \
#	docker-php-ext-enable msgpack



# install APCu
# RUN pecl install apcu && \
#	docker-php-ext-enable apcu --ini-name docker-php-ext-10-apcu.ini

RUN apt-get update -y && apt-get install -y apt-transport-https locales gnupg

# install GD
RUN docker-php-ext-configure gd \
	#	--with-png \
	--with-jpeg \
	--with-xpm \
	--with-webp \
	--with-freetype \
	&& docker-php-ext-install -j$(nproc) gd

# set locale to utf-8
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && locale-gen
ENV LANG='en_US.UTF-8' LANGUAGE='en_US:en' LC_ALL='en_US.UTF-8'



#--------------------------------------------------------------------------
# Final Touches
#--------------------------------------------------------------------------

# install required libs for health check
RUN apt-get -y install libfcgi0ldbl vim htop iotop lsof cron mariadb-client redis-tools

# Set default work directory
# ADD ./scripts/* /usr/local/bin/
RUN chmod +x  /usr/local/bin/*

# Add default configuration files

ADD php.ini /usr/local/etc/php/
ADD xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
ADD www.conf /usr/local/etc/php-fpm.d/

COPY --from=composer /usr/bin/composer /usr/bin/composer


# Health check
RUN echo '#!/bin/bash' > /healthcheck && \
	echo 'env -i SCRIPT_NAME=/health SCRIPT_FILENAME=/health REQUEST_METHOD=GET cgi-fcgi -bind -connect 127.0.0.1:9000 || exit 1' >> /healthcheck && \
	chmod +x /healthcheck

# Clean up
RUN apt-get remove -y git && apt-get autoremove -y && apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN groupadd --gid "${USER_GID}" "${USER}" && \
	useradd --uid ${USER_ID} --gid ${USER_GID} --shell /bin/bash ${USER}

WORKDIR /
