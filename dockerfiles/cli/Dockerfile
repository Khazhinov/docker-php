#####################################################
# Server Side Up -  PHP 8.1 / CLI image
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
    WEBUSER_HOME="/var/www/app" \
    PUID=9999 \
    PGID=9999 \
    LANGUAGE="ru_RU.UTF-8" \
    LANG="ru_RU.UTF-8"

# Install Ondrej repos for Ubuntu focal, PHP8.1, composer and selected extensions
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
        php8.1-cli \
        php8.1-common \
        php8.1-curl \
        php8.1-mbstring \
        php8.1-pgsql \
        php8.1-gd \
        php8.1-redis \
        php8.1-xml \
        php8.1-zip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

# Pull the official composer 2 image, and copy the composer executable from there
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Copy over S6 configurations
COPY etc/cont-init.d/ /etc/cont-init.d/

# Set the default work directory to our web user
WORKDIR /var/www/app

# Configure S6 to drop priveleges
ENTRYPOINT ["/init"]

# Run PHP
CMD ["php", "-a"]
