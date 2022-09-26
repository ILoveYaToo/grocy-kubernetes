FROM --platform=$BUILDPLATFORM php:8.1.8-fpm-alpine as build-extensions
ARG TARGETARCH
RUN apk add --no-cache bash
SHELL ["/bin/bash", "-c"]
RUN apk add --no-cache gd gd-dev libxml2 libxml2-dev sqlite-libs icu icu-dev \
    && if [[ "$TARGETARCH" = "arm64" ]]; then \
           docker-php-ext-configure gd --build x86_64-pc-linux-musl --host aarch64-linux-musl; \
           docker-php-ext-configure simplexml --build x86_64-pc-linux-musl --host aarch64-linux-musl; \
           docker-php-ext-configure intl --build x86_64-pc-linux-musl --host aarch64-linux-musl; \
           docker-php-ext-configure pdo --build x86_64-pc-linux-musl --host aarch64-linux-musl; \
           docker-php-ext-configure pdo_sqlite --build x86_64-pc-linux-musl --host aarch64-linux-musl; \
           docker-php-ext-configure mbstring --build x86_64-pc-linux-musl --host aarch64-linux-musl; \
      	   docker-php-ext-install -j"$(nproc)" gd simplexml intl pdo pdo_sqlite mbstring; \
	         docker-php-ext-enable gd simplexml intl pdo pdo_sqlite mbstring; \
       elif [[ "$TARGETARCH" = "386" ]]; then \
           docker-php-ext-configure gd --build x86_64-pc-linux-musl --host i386-pc-linux-musl; \
           docker-php-ext-configure simplexml --build x86_64-pc-linux-musl --host i386-pc-linux-musl; \
           docker-php-ext-configure intl --build x86_64-pc-linux-musl --host i386-pc-linux-musl; \
           docker-php-ext-configure pdo --build x86_64-pc-linux-musl --host i386-pc-linux-musl; \
           docker-php-ext-configure pdo_sqlite --build x86_64-pc-linux-musl --host i386-pc-linux-musl; \
           docker-php-ext-configure mbstring --build x86_64-pc-linux-musl --host i386-pc-linux-musl; \
	         docker-php-ext-install -j"$(nproc)" gd simplexml intl pdo pdo_sqlite mbstring; \
	         docker-php-ext-enable gd simplexml intl pdo pdo_sqlite mbstring; \
       elif [[ "$TARGETARCH" = "amd64" ]]; then \
	         docker-php-ext-install -j"$(nproc)" opcache gd simplexml intl pdo pdo_sqlite mbstring; \
	         docker-php-ext-enable opcache gd simplexml intl pdo pdo_sqlite mbstring; \
       fi \
    && apk add --no-cache gd libxml2 sqlite-libs icu

FROM php:8.1.8-fpm-alpine as build-pecl
RUN apk add --update --no-cache --virtual .build-dependencies $PHPIZE_DEPS \ 
    && pecl install apcu \
    && docker-php-ext-enable apcu \
    && pecl clear-cache \
    && apk del .build-dependencies 

FROM php:8.1.8-fpm-alpine
ARG APP_VER
ARG TARGETARCH
RUN apk add --no-cache bash
SHELL ["/bin/bash", "-c"]
COPY --from=build-pecl /usr/local/lib/php/extensions/no-debug-non-zts-20210902/ /tmp/pecl_zts_extensions
COPY --from=build-pecl /usr/local/include/php/ext/ /tmp/pecl_include_extensions
COPY --from=build-pecl /usr/local/etc/php/conf.d/ /tmp/pecl_conf
COPY --from=build-extensions /usr/local/lib/php/extensions/no-debug-non-zts-20210902/ /tmp/php_zts_extensions
COPY --from=build-extensions /usr/local/include/php/ext/ /tmp/php_include_extensions
COPY --from=build-extensions /usr/local/etc/php/conf.d/ /tmp/php_conf
RUN cp -R /tmp/pecl_zts_extensions/* /tmp/php_zts_extensions/* /usr/local/lib/php/extensions/no-debug-non-zts-20210902/ \
    && cp -R /tmp/pecl_include_extensions/* /tmp/php_include_extensions/* /usr/local/include/php/ext \
    && cp -R /tmp/pecl_conf/* /tmp/php_conf/* /usr/local/etc/php/conf.d \
    && ls -la /usr/local/etc/php/conf.d
RUN apk add --no-cache wget unzip gd libxml2 sqlite-libs \
    && wget --progress=dot:giga https://github.com/grocy/grocy/archive/refs/tags/v${APP_VER}.zip \
    && unzip -oq v${APP_VER}.zip \
    && mv grocy-${APP_VER}/ /app \
    && apk del wget unzip \
    && mkdir -p /app/data /nginx
RUN adduser -h /app -s /bin/bash -u 1000 -g 1000 -D app \
    && sed -i 's/^user \=.*/user = app/' /usr/local/etc/php-fpm.d/www.conf \
    && sed -i 's/^group \=.*/group = app/' /usr/local/etc/php-fpm.d/www.conf \
    && chown -R app:app /app /nginx
WORKDIR /app
USER app
EXPOSE 9000
CMD ["php-fpm"]
