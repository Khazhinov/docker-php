#####################################################
# Server Side Up -  PHP 8.2 / CLI image
#####################################################

# Default upstream channel to `null` so we default pulling `serversideup/docker-baseimage-s6-overlay-ubuntu:20.04`
# but can also pull the beta image from `serversideup/docker-baseimage-s6-overlay-ubuntu:beta-20.04`
ARG UPSTREAM_CHANNEL=''

FROM serversideup/docker-baseimage-s6-overlay-ubuntu:${UPSTREAM_CHANNEL}20.04

LABEL maintainer="Jay Rogers (@jaydrogers)"
LABEL modified-by="Vladislav Khazhinov (@khazhinov)"

# Make sure we keep apt silent during installs. Set other defaults as well
ENV DEBIAN_FRONTEND=noninteractive \
    S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    WEBUSER_HOME="/app" \
    PUID=9999 \
    PGID=9999 \
    LANGUAGE="ru_RU" \
    LANG="ru_RU.UTF-8"

# Install Ondrej repos for Ubuntu focal, PHP8.2, composer and selected extensions
RUN apt-get update \
    && echo "Добавление пользователя без привилегий..." \
    && groupadd -r -g $PGID webgroup \
    && useradd --no-log-init -r -s /usr/bin/bash -d $WEBUSER_HOME -u $PUID -g $PGID webuser \
    && echo "Установка базовых зависимостей..." \
    && apt-get install -y --no-install-recommends gnupg \
    && echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu focal main" > /etc/apt/sources.list.d/ondrej-php.list \
    && apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4F4EA0AAE5267A6C \
    && apt-get update \
    && apt-get -y --no-install-recommends install \
        ca-certificates \
        curl \
        unzip \
        git \
        make \
        php8.2-dev \
        php8.2-cli \
        php8.2-common \
        php8.2-curl \
        php8.2-mbstring \
        php8.2-pgsql \
        php8.2-gd \
        php8.2-xml \
        php8.2-zip \
        php-pear \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

RUN pecl install redis

COPY conf.d/*.ini /etc/php/8.2/cli/conf.d/

# Pull the official composer 2 image, and copy the composer executable from there
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Copy over S6 configurations
COPY etc/cont-init.d/ /etc/cont-init.d/

# Set the default work directory to our web user
WORKDIR /app

# Configure S6 to drop priveleges
ENTRYPOINT ["/init"]

# Run PHP
CMD ["php", "-a"]
