FROM php:8-fpm-alpine as app

RUN docker-php-ext-install pdo pdo_mysql opcache

CMD ["php-fpm", "--fpm-config", "/usr/local/etc/php-fpm.conf", "--allow-to-run-as-root"]

FROM app as dependencies

COPY --from=composer /usr/bin/composer /usr/bin/composer

ENTRYPOINT ["composer"]