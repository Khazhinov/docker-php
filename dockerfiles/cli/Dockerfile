ARG BASE_IMAGE="ubuntu:22.04"
ARG PHP_VERSION="8.3"
ARG COMPOSER_VERSION="2.7.7"
ARG APPLICATION_WORKDIR="/app"

####################### Хак синтаксиса Dockerfile для обеспечения версионности засчёт аргументов
FROM composer:${COMPOSER_VERSION} as composer
ARG BASE_IMAGE="ubuntu:22.04"
ARG PHP_VERSION="8.2"
ARG COMPOSER_VERSION="2.6.6"
ARG APPLICATION_WORKDIR="/app"

####################### Базовая установка образа с S6 Overlay
FROM ${BASE_IMAGE}
ARG BASE_IMAGE
ARG PHP_VERSION
ARG COMPOSER_VERSION
ARG APPLICATION_WORKDIR

ENV DEBIAN_FRONTEND=noninteractive \
    COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_HOME=/composer \
    COMPOSER_MAX_PARALLEL_HTTP=24 \
    PHP_VERSION=${PHP_VERSION}

LABEL maintainer="Vladislav Khazhinov (@khazhinov)"

# Установка базовых зависимостей
RUN echo "Установка базовых зависимостей..." \
    && apt-get update \
    && apt-get install -y --no-install-recommends unzip git make xz-utils gnupg2 ca-certificates software-properties-common curl;

# Установка зависимостей PHP
RUN echo "Добавление репозитория PHP..." \
    && add-apt-repository -y ppa:ondrej/php \
    && apt-get update \
    ; \
    echo "Установка окружения PHP с предустановленным pecl для сборки расширений..." \
    && apt-get -y --no-install-recommends install \
        php${PHP_VERSION}-cli \
        php${PHP_VERSION}-common \
        php${PHP_VERSION}-dev \
        php-pear \
        \
        php${PHP_VERSION}-curl \
        php${PHP_VERSION}-mbstring \
        php${PHP_VERSION}-gd \
        php${PHP_VERSION}-xml \
        php${PHP_VERSION}-bcmath \
        php${PHP_VERSION}-intl \
        php${PHP_VERSION}-pgsql \
        php${PHP_VERSION}-redis \
        php${PHP_VERSION}-zip \
    ; \
    echo "Установка символической ссылки для PHP ${PHP_VERSION}..." \
    && ln -sf /etc/php/${PHP_VERSION}/ /etc/php/current_version \
    ; \
    echo "Очистка системных кешей..." \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

# Установка пусконаладочных скриптов
COPY --chmod=+x docker-php-* /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-php-*

# Установка конфигурационных файлов
COPY ./conf.d/ ${PHP_INI_DIR}/conf.d/

# Применяем описанный в начале хак. Здесь composer ссылается не на composer:latest, а на указанный в аргументах
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Финальный вывод информации о сборке
RUN set -eux; \
    php -v; \
    php -m; \
    composer --version;

# Установка workdir
WORKDIR ${APPLICATION_WORKDIR}

ENTRYPOINT ["docker-php-entrypoint"]
CMD ["php", "-a"]