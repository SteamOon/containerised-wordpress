FROM	debian:bookworm-slim

ARG	PHPPATH=/etc/php/8.2/fpm

# install PHP & extensions
RUN 	apt-get update && apt-get install --no-install-recommends --no-install-suggests -y \
	ca-certificates \
	php8.2-fpm \
	php8.2-mysql \
	php8.2-curl \
	php8.2-gd \
	php8.2-intl \
	php8.2-mbstring \
	php8.2-soap \
	php8.2-xml \
	php8.2-zip \
	wget \
	tar \
	&& ln -s /usr/sbin/php-fpm8.2 /usr/local/bin/php-fpm \
	&& rm -rf /var/lib/apt/lists/* \
	&& service php8.2-fpm stop

COPY	conf/www.conf ${PHPPATH}/pool.d/

RUN	sed -i 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' ${PHPPATH}/php.ini && \
	sed -i "s/listen = \/run\/php\/php$PHP_VERSION_ENV-fpm.sock/listen = 9000/g" ${PHPPATH}/pool.d/www.conf && \
	sed -i 's/;listen.mode = 0660/listen.mode = 0660/g' ${PHPPATH}/pool.d/www.conf && \
	sed -i 's/;daemonize = yes/daemonize = no/g' ${PHPPATH}/pool.d/www.conf

RUN	wget --no-check-certificate https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar && \
	chmod +x wp-cli.phar && \
	mv wp-cli.phar /usr/local/bin/wp

RUN	mkdir -p /run/php/ && \
	mkdir -p /var/run/php/ && \
	mkdir -p /var/www/inception/

RUN	chown -R www-data:www-data /var/www/inception/

COPY	conf/wp-config.php /var/www/inception/

COPY	tools/wp_setup.sh /bin/

RUN	chmod +x /bin/wp_setup.sh

EXPOSE 9000

CMD		["wp_setup.sh", "php-fpm8.2", "--nodaemonize"]
