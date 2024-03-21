FROM php:8-fpm-alpine as app

RUN docker-php-ext-install pdo pdo_mysql opcache

CMD ["php-fpm", "--fpm-config", "/usr/local/etc/php-fpm.conf", "--allow-to-run-as-root"]

FROM app as debug

RUN apk add $PHPIZE_DEPS
RUN apk add --update linux-headers
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug

FROM app as dependencies

RUN apk add --update git
COPY --from=composer /usr/bin/composer /usr/bin/composer

ENTRYPOINT ["composer"]