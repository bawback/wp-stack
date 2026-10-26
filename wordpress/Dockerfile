FROM wordpress:6.5-php8.2-apache

# PHP pgsql extension + PG4WP plugin (lets WordPress talk to PostgreSQL)
RUN apt-get update \
 && apt-get install -y --no-install-recommends libpq-dev git \
 && docker-php-ext-install pgsql \
 && git clone --depth 1 https://github.com/PostgreSQL-For-Wordpress/postgresql-for-wordpress.git /tmp/pg4wp \
 && cp -r /tmp/pg4wp/pg4wp /usr/src/wordpress/wp-content/pg4wp \
 && cp /usr/src/wordpress/wp-content/pg4wp/db.php /usr/src/wordpress/wp-content/db.php \
 && rm -rf /tmp/pg4wp \
 && apt-get purge -y git && apt-get autoremove -y && rm -rf /var/lib/apt/lists/*