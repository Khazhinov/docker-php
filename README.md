
<p align="center"><img src="/art/header.png" alt="Social Card of Docker PHP"></p>

# Docker PHP 🐋

Сборки PHP, оптимизированные для работы в Production

## Описание

Данные docker контейнеры предназначены для быстрого разворачивания php приложений. Все сборки по-умолчанию включат в себя всё необходимое для запуска кода на любом современном фреймворке (Larave, Symfony, etc.).

## Примеры

Рассмотрим пример с запуском Laravel приложения для минимального Production окружения. Для обработки HTTP запросов будем использовать Roadrunner, для планировщика задач и очередей будем использовать базовую CLI сборку.

В корне Laravel проекта создадим файл ```API.Dockerfile``` для обработки HTTP со следующим содержанием:

```dockerfile
# Базовый образ RouaRunner: https://roadrunner.dev/
FROM ghcr.io/roadrunner-server/roadrunner:latest AS roadrunner
# Базовый образ: https://github.com/khazhinov/docker-php
FROM khazhinov/docker-php:8.1-cli

## Пример для установки дополнительных расширений PHP
#RUN apt-get update \
#    && apt-get install -y --no-install-recommends php8.1-imagick \
#    && apt-get clean \
#    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

COPY . /var/www/app

COPY --from=roadrunner /usr/bin/rr /usr/local/bin/rr
COPY --from=roadrunner /usr/bin/rr /var/www/app/rr

#RUN chown -R webuser: /var/www/app
RUN chmod -R 777 /var/www/app/storage /var/www/app/bootstrap/cache

CMD ["su", "webuser", "-c", "php artisan octane:roadrunner --host=0.0.0.0 --port=80 --rpc-port=6001 --rr-config=.rr.api.yaml"]
```

Также создадим ```CLI.Dockerfile``` для запуска планировщика и очередей:

```dockerfile
# Базовый образ: https://github.com/khazhinov/docker-php
FROM khazhinov/docker-php:8.1-cli

## Пример для установки дополнительных расширений PHP
#RUN apt-get update \
#    && apt-get install -y --no-install-recommends php8.1-pgsql \
#    && apt-get clean \
#    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

COPY . /var/www/app

RUN chown -R webuser: /var/www/app
RUN chmod -R 777 /var/www/app/storage /var/www/app/bootstrap/cache

```

Для запуска окружения создадим ```docker-compose.yaml```:

```yaml
version: '3.9'

networks:
  my-project-network:
    name: my-project-network
    driver: bridge

services:
  api:
    build:
      context: .
      dockerfile: API.Dockerfile
    container_name: my-project-api
    restart: always
    command: ["su", "webuser", "-c", "php artisan octane:roadrunner --host=0.0.0.0 --port=80 --rpc-port=6001 --rr-config=.rr.api.yaml"]
    ports: ["8990:80"]
    environment:
      PHP_POOL_NAME: "my-project-api"
    networks:
      - my-project-network
    depends_on:
      - redis
      - postgres

  task:
    build:
      context: .
      dockerfile: CLI.Dockerfile
    container_name: my-project-task
    restart: always
    command: ["su", "webuser", "-c", "php artisan schedule:work"]
    environment:
      PHP_POOL_NAME: "my-project-task"
    networks:
      - my-project-network
    depends_on:
      - redis
      - postgres

  queue:
    build:
      context: .
      dockerfile: CLI.Dockerfile
    container_name: my-project-queue
    restart: always
    command: ["su", "webuser", "-c", "php artisan queue:work --tries=3"]
    environment:
      PHP_POOL_NAME: "my-project-queue"
    networks:
      - my-project-network
    depends_on:
      - redis
      - postgres

  redis:
    image: redislabs/rejson:2.0.7
    container_name: my-project-redis
    restart: always
    ports: ["6369:6379"]
    volumes:
      - ./docker/data/redis/database:/data
    networks:
      - my-project-network

  postgres:
    image: postgres:12.9-alpine
    container_name: my-project-postgres
    restart: always
    ports: ["5492:5432"]
    environment:
      POSTGRES_DB: my-project
      POSTGRES_USER: my-project
      POSTGRES_PASSWORD: my-project
    volumes:
      - ./docker/data/postgresql/database:/var/lib/postgresql/data
      - ./docker/data/postgresql/home:/root
    networks:
      - my-project-network
```

## Благодарности ❤️

Отдельная благодарность разработчикам [serversideup/docker-baseimage-s6-overlay-ubuntu](https://github.com/serversideup/docker-baseimage-s6-overlay-ubuntu), чей контейнер используется в качестве базового.

## Лицензия

Лицензия MIT. Для получения большей информации обращайтесь к [тексту лицензии](LICENSE.md).
