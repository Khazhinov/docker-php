# Базовый образ с супервизором процессов ServerSideUp: https://hub.docker.com/r/serversideup/s6-overlay
ARG UPSTREAM_CHANNEL=''
ARG BASE_OS_FLAVOR='ubuntu'
ARG BASE_OS_VERSION='22.04'
ARG S6_OVERLAY_VERSION='v3.1.4.2'
ARG BASE_IMAGE="serversideup/s6-overlay:${UPSTREAM_CHANNEL}${BASE_OS_FLAVOR}-${BASE_OS_VERSION}-${S6_OVERLAY_VERSION}"

# Установка репозиториев PHP
FROM ${BASE_IMAGE} as repo-config

RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg2 ca-certificates software-properties-common \
    && add-apt-repository -y ppa:ondrej/php

# Сборка результирующего образа
FROM ${BASE_IMAGE}

LABEL maintainer="Jay Rogers (@jaydrogers)"
LABEL modified-by="Vladislav Khazhinov (@khazhinov)"

ARG PHP_VERSION='8.2'

ENV BUILD_PHP_VERSION=$PHP_VERSION \
    DEBIAN_FRONTEND=noninteractive \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    S6_VERBOSITY=1 \
    COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_HOME=/composer \
    COMPOSER_MAX_PARALLEL_HTTP=24 \
    WEBUSER_HOME="/app" \
    PUID=9999 \
    PGID=9999

COPY --from=repo-config /etc/apt/sources.list.d/ /etc/apt/sources.list.d/
COPY --from=repo-config /etc/apt/trusted.gpg.d/ondrej-ubuntu-php.gpg /etc/apt/trusted.gpg.d/ondrej-ubuntu-php.gpg

RUN apt-get update \
    \
    && echo "Создание пользователя без root доступа..." \
    && groupadd -r -g $PGID webgroup \
    && useradd --no-log-init -r -s /usr/bin/bash -d $WEBUSER_HOME -u $PUID -g $PGID webuser \
    \
    && echo "Установка базовых зависимостей..." \
    && apt-get -y --no-install-recommends install \
        ca-certificates \
        curl \
        unzip \
        git \
        make \
    \
    && echo "Установка окружения PHP с предустановленным pecl для сборки расширений..." \
    && apt-get update \
    && apt-get -y --no-install-recommends install \
        php8.2-cli \
        php8.2-common \
        php8.2-dev \
        php-pear \
        \
        php8.2-curl \
        php8.2-mbstring \
        php8.2-gd \
        php8.2-xml \
        php8.2-bcmath \
        php8.2-intl \
        php8.2-pgsql \
        php8.2-redis \
        php8.2-zip \
    \
    && echo "Установка символической ссылки..." \
    && ln -sf /etc/php/8.2/ /etc/php/current_version \
    \
    && echo "Очистка системных кешей..." \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

COPY --from=composer:2.5.8 /usr/bin/composer /usr/bin/composer

COPY --chmod=755 etc/s6-overlay/ /etc/s6-overlay/

COPY conf.d/*.ini /etc/php/current_version/cli/conf.d/

WORKDIR /app

ENTRYPOINT ["/init"]

CMD ["php", "-a"]