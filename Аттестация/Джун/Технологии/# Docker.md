**## Что это**

  

Docker — это открытая платформа для разработки, доставки и запуска приложений в изолированных средах, называемых контейнерами. Docker позволяет упаковать приложение со всеми его зависимостями (библиотеки, конфигурации, системные инструменты) в единый образ, который можно запустить на любой системе с Docker.

  

**### История и предпосылки**

  

Docker был представлен в 2013 году компанией dotCloud (позже переименованной в Docker, Inc.). Платформа быстро стала стандартом индустрии для контейнеризации приложений. До Docker существовали другие технологии контейнеризации (LXC, FreeBSD Jails), но именно Docker сделал контейнеры доступными и простыми в использовании для широкой аудитории разработчиков.

  

**### Основные проблемы, которые решает Docker**

  

**1. "Работает на моей машине"**

Классическая проблема, когда приложение работает на машине разработчика, но не работает на сервере или у другого разработчика. Docker решает это, обеспечивая одинаковое окружение везде.

  

**2. Сложность настройки окружения**

Вместо установки и настройки всех зависимостей вручную, можно запустить готовый образ с уже настроенным окружением.

  

**3. Конфликты зависимостей**

Разные приложения могут требовать разные версии одних и тех же библиотек. Контейнеры изолируют зависимости каждого приложения.

  

**4. Эффективное использование ресурсов**

Контейнеры более легковесны, чем виртуальные машины, позволяя запускать больше приложений на том же оборудовании.

  

**5. Масштабирование**

Легко создавать множество копий приложения для обработки увеличенной нагрузки.

  

**### Архитектура Docker**

  

Docker использует клиент-серверную архитектуру:

  

**Docker Client (клиент)**

• Интерфейс командной строки (`docker`)

• Отправляет команды Docker демону

• Может подключаться к удаленным демонам

  

**Docker Daemon (демон/сервер)**

• Фоновый процесс (`dockerd`)

• Управляет образами, контейнерами, сетями и volumes

• Слушает Docker API запросы

• Общается с другими демонами для управления Docker сервисами

  

**Docker Registry (реестр)**

• Хранилище Docker образов

• Docker Hub — публичный реестр (hub.docker.com)

• Можно создавать приватные реестры

• Содержит официальные образы популярных приложений

  

**Docker Objects (объекты Docker):**

• Images (образы) — шаблоны для создания контейнеров

• Containers (контейнеры) — запущенные экземпляры образов

• Networks (сети) — связи между контейнерами

• Volumes (тома) — постоянное хранилище данных

  

**### Основные преимущества Docker**

  

**1. Консистентность окружения**

• Одинаковая работа на разработке, тестировании и продакшене

• Устранение проблем "works on my machine"

• Воспроизводимые сборки

  

**2. Изоляция приложений**

• Каждый контейнер работает изолированно

• Конфликты зависимостей исключены

• Повышенная безопасность через изоляцию

  

**3. Скорость и эффективность**

• Быстрый запуск (секунды, не минуты)

• Эффективное использование ресурсов

• Меньше overhead по сравнению с VM

  

**4. Портативность**

• Запуск на любой системе с Docker

• Легкая миграция между облачными провайдерами

• Одинаковая работа на Windows, macOS, Linux

  

**5. Масштабируемость**

• Легко создавать множество экземпляров

• Горизонтальное масштабирование

• Интеграция с оркестраторами (Kubernetes, Docker Swarm)

  

**6. DevOps и CI/CD**

• Упрощение процессов развертывания

• Интеграция с CI/CD пайплайнами

• Быстрые откаты к предыдущим версиям

  

**7. Экосистема**

• Огромное количество готовых образов на Docker Hub

• Активное сообщество

• Богатая документация и инструменты

  

**### Основные концепции и термины**

  

**Image (Образ)**

• Read-only шаблон для создания контейнеров

• Содержит ОС, приложение и все зависимости

• Состоит из слоев (layers)

• Создается из Dockerfile или commit контейнера

  

**Container (Контейнер)**

• Запущенный экземпляр образа

• Изолированная среда выполнения

• Может быть запущен, остановлен, удален

• Имеет собственную файловую систему, процессы, сеть

  

**Dockerfile**

• Текстовый файл с инструкциями для сборки образа

• Описывает шаги создания образа

• Версионируется вместе с кодом

• Автоматизирует создание образов

  

**Layer (Слой)**

• Каждая инструкция в Dockerfile создает слой

• Слои кэшируются и переиспользуются

• Неизменяемые (immutable)

• Экономят место и время сборки

  

**Volume (Том)**

• Механизм постоянного хранения данных

• Данные переживают удаление контейнера

• Может быть разделен между контейнерами

• Управляется Docker

  

**Network (Сеть)**

• Позволяет контейнерам общаться друг с другом

• Изолирует трафик между группами контейнеров

• Несколько типов: bridge, host, overlay, none  
  
🧱 **1. bridge**

• **По умолчанию** для контейнеров на одном хосте.

• Создаёт **виртуальный мост** (обычно docker0), через который контейнеры общаются между собой и с хостом.

• Можно назначать свои подсети, порты, алиасы.

🔹 _Используется чаще всего.  
  
_🌐 **2. host**

• Контейнер **использует сетевой стек хоста напрямую** — без изоляции.

• IP контейнера = IP хоста.

• Нет проброса портов (порты контейнера = порты хоста).

🔹 _Максимальная производительность, но нет сетевой изоляции.  
  
_🧩 **3. overlay**

• Для **мультихостовых** сетей (Docker Swarm, Kubernetes и т.д.).

• Создаёт **виртуальную сеть поверх нескольких хостов**.

• Контейнеры на разных серверах могут общаться как будто в одной сети.

🔹 _Используется в распределённых кластерах._

  
🚫 **4. none**

• Контейнер **без сети вообще**.

• Нет IP, нет доступа к внешнему миру, только localhost внутри.

🔹 _Используется для изоляции или кастомных сетевых настроек._

  

**Registry (Реестр)**

• Хранилище и дистрибуция образов

• Docker Hub — публичный реестр

• Можно развернуть приватный реестр

• Поддержка версионирования через теги

  

**### Основные команды Docker**

  

**Работа с образами:**

```bash

# Скачать образ из реестра

docker pull nginx:latest

  

# Список локальных образов

docker images

  

# Построить образ из Dockerfile

docker build -t myapp:1.0 .

  

# Удалить образ

docker rmi nginx:latest

  

# Просмотр истории образа

docker history nginx

  

# Поиск образов в Docker Hub

docker search nginx

  

# Сохранить образ в файл

docker save -o myapp.tar myapp:1.0

  

# Загрузить образ из файла

docker load -i myapp.tar

  

# Пометить образ новым тегом

docker tag myapp:1.0 myapp:latest

```

  

**Работа с контейнерами:**

```bash

# Запустить контейнер

docker run -d --name mycontainer nginx

  

# Запустить с пробросом портов

docker run -d -p 8080:80 nginx

  

# Запустить с volume

docker run -d -v /host/path:/container/path nginx

  

# Запустить с переменными окружения

docker run -d -e "VAR=value" nginx

  

# Список запущенных контейнеров

docker ps

  

# Список всех контейнеров (включая остановленные)

docker ps -a

  

# Остановить контейнер

docker stop mycontainer

  

# Запустить остановленный контейнер

docker start mycontainer

  

# Перезапустить контейнер

docker restart mycontainer

  

# Удалить контейнер

docker rm mycontainer

  

# Удалить запущенный контейнер (force)

docker rm -f mycontainer

  

# Просмотр логов

docker logs mycontainer

docker logs -f mycontainer  # следить за логами

  

# Выполнить команду в контейнере

docker exec -it mycontainer bash

  

# Просмотр использования ресурсов

docker stats

  

# Информация о контейнере

docker inspect mycontainer

  

# Копирование файлов в/из контейнера

docker cp file.txt mycontainer:/path/

docker cp mycontainer:/path/file.txt .

```

  

**Очистка:**

```bash

# Удалить все остановленные контейнеры

docker container prune

  

# Удалить неиспользуемые образы

docker image prune

  

# Удалить все неиспользуемые ресурсы

docker system prune

  

# Удалить все (включая volumes)

docker system prune -a --volumes

```

  

**### Пример Dockerfile**

  

**Простой PHP приложение:**

```dockerfile

# Базовый образ

FROM php:8.2-apache

  

# Метаданные

LABEL maintainer="dev@example.com"

LABEL version="1.0"

LABEL description="PHP Application"

  

# Установка системных зависимостей

RUN apt-get update && apt-get install -y \

    git \

    curl \

    libpng-dev \

    libonig-dev \

    libxml2-dev \

    zip \

    unzip

  

# Очистка кэша

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

  

# Установка PHP расширений

RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

  

# Установка Composer

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

  

# Установка рабочей директории

WORKDIR /var/www/html

  

# Копирование файлов приложения

COPY . /var/www/html

  

# Установка зависимостей

RUN composer install --no-dev --optimize-autoloader

  

# Настройка прав доступа

RUN chown -R www-data:www-data /var/www/html \

    && chmod -R 755 /var/www/html/storage

  

# Открытие порта

EXPOSE 80

  

# Команда запуска

CMD ["apache2-foreground"]

```

  

**Node.js приложение:**

```dockerfile

FROM node:18-alpine

  

# Создание директории приложения

WORKDIR /app

  

# Копирование package.json и package-lock.json

COPY package*.json ./

  

# Установка зависимостей

RUN npm ci --only=production

  

# Копирование исходников

COPY . .

  

# Сборка приложения

RUN npm run build

  

# Использование непривилегированного пользователя

USER node

  

# Открытие порта

EXPOSE 3000

  

# Healthcheck

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \

  CMD node healthcheck.js

  

# Запуск приложения

CMD ["node", "dist/main.js"]

```

  

**### Best Practices для Docker**

  

**1. Используйте официальные базовые образы**

```dockerfile

# Хорошо

FROM node:18-alpine

  

# Плохо - неофициальный образ

FROM someuser/node

```

  

**2. Используйте .dockerignore**

```

node_modules

npm-debug.log

.git

.env

*.md

.DS_Store

```

  

**3. Минимизируйте количество слоев**

```dockerfile

# Хорошо - один слой

RUN apt-get update && apt-get install -y \

    package1 \

    package2 \

    && rm -rf /var/lib/apt/lists/*

  

# Плохо - три слоя

RUN apt-get update

RUN apt-get install -y package1

RUN apt-get install -y package2

```

  

**4. Используйте multi-stage builds**

```dockerfile

# Стадия сборки

FROM node:18 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

  

# Продакшн стадия

FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/dist ./dist

COPY --from=builder /app/node_modules ./node_modules

CMD ["node", "dist/main.js"]

```

  

**5. Не запускайте процессы от root**

```dockerfile

RUN addgroup -g 1001 -S nodejs

RUN adduser -S nodejs -u 1001

USER nodejs

```

  

**6. Используйте конкретные версии**

```dockerfile

# Хорошо

FROM node:18.16.0-alpine

  

# Плохо

FROM node:latest

```

  

**7. Кэшируйте зависимости**

```dockerfile

# Сначала копируем только файлы зависимостей

COPY package*.json ./

RUN npm install

  

# Потом копируем остальное

COPY . .

```

  

**### Use Cases (Сценарии использования)**

  

**1. Микросервисы**

• Каждый сервис в отдельном контейнере

• Независимое развертывание и масштабирование

• Изоляция сбоев

  

**2. Разработка**

• Быстрая настройка окружения разработки

• Одинаковое окружение для всей команды

• Легкое переключение между проектами

  

**3. CI/CD**

• Консистентные сборки

• Быстрое тестирование

• Автоматизированное развертывание

  

**4. Тестирование**

• Изолированные тестовые окружения

• Параллельное выполнение тестов

• Легкая очистка после тестов

  

**5. Миграция в облако**

• Портативность между облачными провайдерами

• Гибридные облачные архитектуры

• Локальное тестирование облачных приложений

  

**## Что такое контейнер**

  

Контейнер — это стандартизированная единица программного обеспечения, которая упаковывает код приложения и все его зависимости, чтобы приложение быстро и надежно работало в любом вычислительном окружении. Это изолированная среда выполнения, которая содержит всё необходимое для работы приложения: код, runtime, системные инструменты, системные библиотеки и настройки.

  

**### Технические основы контейнеров**

  

Контейнеры построены на технологиях ядра Linux:

  

**Linux Namespaces (пространства имен)**

Обеспечивают изоляцию на уровне системы:

• **PID** — изоляция процессов (каждый контейнер видит только свои процессы)

• **Network** — изоляция сети (свой IP, порты, таблицы маршрутизации)

• **Mount** — изоляция файловой системы (своя файловая система)

• **IPC** — изоляция межпроцессного взаимодействия

• **UTS** — изоляция hostname и domain name

• **User** — изоляция пользователей и групп

• **Cgroup** — изоляция ресурсов и их ограничение

  

**Control Groups (cgroups)**

Управляют и ограничивают ресурсы:

• Ограничение CPU (количество ядер, доля времени)

• Ограничение памяти (RAM, swap)

• Ограничение дисковых операций (I/O)

• Ограничение сетевой пропускной способности

• Учет использования ресурсов

  

**Union File Systems (UnionFS)**

Позволяют создавать слои файловой системы:

• Образ состоит из read-only слоев

• Контейнер добавляет writable слой сверху

• Эффективное использование места

• Быстрое создание контейнеров

• Примеры: AUFS, OverlayFS, Btrfs, ZFS

  

**### Жизненный цикл контейнера**

  

**1. Created (Создан)**

```bash

docker create --name myapp nginx

```

Контейнер создан, но не запущен. Файловая система подготовлена, но процессы не запущены.

  

**2. Running (Запущен)**

```bash

docker start myapp

# или сразу создать и запустить

docker run --name myapp nginx

```

Контейнер активен, процессы выполняются.

  

**3. Paused (Приостановлен)**

```bash

docker pause myapp

```

Процессы заморожены, но контейнер остается в памяти. Используется для временной приостановки без остановки.

  

**4. Stopped (Остановлен)**

```bash

docker stop myapp  # graceful stop (SIGTERM)

docker kill myapp  # force stop (SIGKILL)

```

Процессы завершены, но файловая система контейнера сохранена.

  

**5. Deleted (Удален)**

```bash

docker rm myapp

```

Контейнер и его writable layer удалены. Данные в volumes остаются.

  

**### Ключевые характеристики контейнеров**

  

**1. Легковесность**

• Размер образа: от нескольких MB (Alpine Linux ~5MB)

• Запуск: менее секунды

• Использует ядро хост-системы

• Не требует полной ОС

• Можно запустить сотни контейнеров на одном хосте

  

**2. Изоляция**

• Процессы не видят друг друга

• Отдельная файловая система для каждого контейнера

• Изолированная сеть (если не указано иное)

• Ограничение ресурсов (CPU, память, I/O)

• Повышенная безопасность

  

**3. Портативность**

• "Build once, run anywhere"

• Одинаковая работа на любой платформе с Docker

• Независимость от хост-системы

• Легкая миграция между серверами

• Работа в облаке и on-premise

  

**4. Эфемерность (Ephemeral)**

• Контейнеры предназначены для временного использования

• Данные не сохраняются после удаления (без volumes)

• Легко создавать и удалять

• Immutable infrastructure подход

• Cattle, not pets философия

  

**5. Иммутабельность**

• Образ неизменяем после создания

• Изменения в контейнере не влияют на образ

• Для обновления создается новый образ

• Предсказуемое поведение

• Легкие откаты к предыдущим версиям

  

**6. Масштабируемость**

• Быстрое создание множества экземпляров

• Горизонтальное масштабирование

• Автоматическое масштабирование (с оркестраторами)

• Load balancing между контейнерами

  

**### Отличия контейнеров от виртуальных машин**

  

**Архитектура:**

  

**Виртуальная машина:**

```

[Приложение A] [Приложение B]

[Гостевая ОС]   [Гостевая ОС]

[Гипервизор]

[Хост ОС]

[Физическое оборудование]

```

  

**Контейнер:**

```

[Приложение A] [Приложение B]

[Docker Engine]

[Хост ОС]

[Физическое оборудование]

```

  

**Сравнение характеристик:**

  

| Характеристика | Контейнер | Виртуальная машина |

|---|---|---|

| **Размер** | Мегабайты | Гигабайты |

| **Время запуска** | Секунды | Минуты |

| **Изоляция** | На уровне процессов | Полная изоляция |

| **Производительность** | Близка к native | Есть overhead |

| **ОС** | Общее ядро | Отдельная ОС |

| **Плотность** | Сотни на хосте | Десятки на хосте |

| **Портативность** | Высокая | Средняя |

  

**Когда использовать контейнеры:**

• Микросервисы и cloud-native приложения

• CI/CD пайплайны

• Быстрое масштабирование

• Эффективное использование ресурсов

• Современные приложения

  

**Когда использовать VM:**

• Нужна полная изоляция

• Разные операционные системы

• Legacy приложения

• Высокие требования к безопасности

• Stateful приложения

  

**Гибридный подход:**

Часто используется комбинация: VM для изоляции окружений, контейнеры внутри VM для приложений.

  

**### Типы контейнеров**

  

**1. Application Containers**

• Содержат одно приложение

• Обычно один процесс

• Примеры: веб-сервер, API, база данных

  

**2. System Containers**

• Ведут себя как полноценные системы

• Несколько процессов

• Init система (systemd)

• Примеры: LXC/LXD контейнеры

  

**3. Sidecar Containers**

• Дополняют основной контейнер

• Общий namespace с основным контейнером

• Примеры: логирование, мониторинг, прокси

  

**4. Init Containers**

• Запускаются до основного контейнера

• Используются для инициализации

• Примеры: миграции БД, настройка конфигурации

  

**### Безопасность контейнеров**

  

**Лучшие практики:**

  

**1. Используйте минимальные базовые образы**

```dockerfile

# Alpine Linux - минимальный дистрибутив

FROM alpine:3.18

  

# Distroless - без package manager и shell

FROM gcr.io/distroless/nodejs18

```

  

**2. Не запускайте от root**

```dockerfile

RUN adduser -D appuser

USER appuser

```

  

**3. Сканируйте образы на уязвимости**

```bash

docker scan myapp:latest

trivy image myapp:latest

```

  

**4. Используйте read-only filesystem**

```bash

docker run --read-only myapp

```

  

**5. Ограничивайте capabilities**

```bash

docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

```

  

**6. Используйте секреты правильно**

```bash

# НЕ включайте секреты в образ

# Используйте Docker secrets или переменные окружения

docker run -e DB_PASSWORD=$DB_PASSWORD myapp

```

  

**7. Ограничивайте ресурсы**

```bash

docker run --memory=512m --cpus=1 myapp

```

  

**8. Регулярно обновляйте образы**

```bash

docker pull node:18-alpine

docker build --no-cache -t myapp:latest .

```

  

**### Networking в контейнерах**

  

**Типы сетей:**

  

**1. Bridge (по умолчанию)**

• Виртуальная сеть на хосте

• Контейнеры могут общаться друг с другом

• NAT для доступа во внешний мир

```bash

docker network create my-bridge

docker run --network my-bridge nginx

```

  

**2. Host**

• Контейнер использует сеть хоста напрямую

• Максимальная производительность

• Нет изоляции портов

```bash

docker run --network host nginx

```

  

**3. Overlay**

• Для связи контейнеров на разных хостах

• Используется в Docker Swarm и Kubernetes

• Требует key-value store

  

**4. Macvlan**

• Контейнер получает MAC адрес

• Выглядит как физическое устройство в сети

• Для legacy приложений

  

**5. None**

• Полная изоляция сети

• Только loopback интерфейс

```bash

docker run --network none nginx

```

  

**Проброс портов:**

```bash

# Проброс одного порта

docker run -p 8080:80 nginx

  

# Проброс на конкретный интерфейс

docker run -p 127.0.0.1:8080:80 nginx

  

# Случайный порт хоста

docker run -p 80 nginx

  

# Несколько портов

docker run -p 80:80 -p 443:443 nginx

```

  

**### Storage в контейнерах**

  

**Типы хранилищ:**

  

**1. Volumes (рекомендуется)**

```bash

# Создать volume

docker volume create mydata

  

# Использовать volume

docker run -v mydata:/data nginx

  

# Список volumes

docker volume ls

  

# Удалить volume

docker volume rm mydata

```

  

**2. Bind Mounts**

```bash

# Монтировать директорию хоста

docker run -v /host/path:/container/path nginx

  

# Read-only mount

docker run -v /host/path:/container/path:ro nginx

```

  

**3. tmpfs (в памяти)**

```bash

docker run --tmpfs /tmp nginx

```

  

**Сравнение:**

  

| Тип | Управление | Производительность | Use Case |

|---|---|---|---|

| Volume | Docker | Высокая | Продакшн данные |

| Bind Mount | Пользователь | Средняя | Разработка |

| tmpfs | Автоматически | Очень высокая | Временные данные |

  

**### Мониторинг и логирование**

  

**Просмотр ресурсов:**

```bash

# Использование ресурсов всех контейнеров

docker stats

  

# Конкретный контейнер

docker stats mycontainer

  

# Детальная информация

docker inspect mycontainer

```

  

**Логи:**

```bash

# Просмотр логов

docker logs mycontainer

  

# Следить за логами в реальном времени

docker logs -f mycontainer

  

# Последние N строк

docker logs --tail 100 mycontainer

  

# С временными метками

docker logs -t mycontainer

  

# За период времени

docker logs --since 2024-01-01 mycontainer

```

  

**Logging drivers:**

```bash

# json-file (по умолчанию)

docker run --log-driver json-file nginx

  

# syslog

docker run --log-driver syslog nginx

  

# journald

docker run --log-driver journald nginx

  

# gelf (Graylog)

docker run --log-driver gelf \

  --log-opt gelf-address=udp://graylog:12201 nginx

  

# Отключить логи

docker run --log-driver none nginx

```

  

**## Что такое docker-compose**

  

Docker Compose — это инструмент для определения и управления многоконтейнерными Docker-приложениями. С помощью Compose вы используете YAML файл для конфигурации сервисов вашего приложения, а затем одной командой создаете и запускаете все сервисы из этой конфигурации.

  

**### Зачем нужен Docker Compose**

  

**Проблемы без Compose:**

```bash

# Запуск веб-приложения вручную:

docker network create myapp-network

docker volume create db-data

docker run -d --name db --network myapp-network \

  -v db-data:/var/lib/mysql \

  -e MYSQL_ROOT_PASSWORD=secret \

  mysql:8.0

docker run -d --name redis --network myapp-network redis:7

docker run -d --name web --network myapp-network \

  -p 80:80 \

  -e DB_HOST=db \

  -e REDIS_HOST=redis \

  myapp:latest

```

  

**С Compose:**

```bash

docker-compose up -d

```

  

**### Установка Docker Compose**

  

**Docker Compose V2 (рекомендуется):**

Входит в состав Docker Desktop и современных версий Docker Engine.

```bash

docker compose version

```

  

**Docker Compose V1 (legacy):**

```bash

docker-compose version

```

  

**### Структура docker-compose.yml**

  

**Базовый формат:**

```yaml

version: '3.8'  # версия формата файла

  

services:       # определение сервисов

  service1:

    # конфигурация сервиса

  service2:

    # конфигурация сервиса

  

networks:       # определение сетей (опционально)

  network1:

  

volumes:        # определение volumes (опционально)

  volume1:

  

configs:        # конфигурационные файлы (опционально)

  config1:

  

secrets:        # секреты (опционально)

  secret1:

```

  

**### Полный пример веб-приложения**

  

**docker-compose.yml:**

```yaml

version: '3.8'

  

services:

  # Веб-сервер Nginx

  nginx:

    image: nginx:1.25-alpine

    container_name: myapp-nginx

    restart: unless-stopped

    ports:

      - "80:80"

      - "443:443"

    volumes:

      - ./nginx/conf.d:/etc/nginx/conf.d:ro

      - ./nginx/ssl:/etc/nginx/ssl:ro

      - web-static:/var/www/html/static

    depends_on:

      - app

    networks:

      - frontend

      - backend

    healthcheck:

      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/health"]

      interval: 30s

      timeout: 10s

      retries: 3

      start_period: 40s

  

  # PHP-FPM приложение

  app:

    build:

      context: ./app

      dockerfile: Dockerfile

      args:

        - PHP_VERSION=8.2

    container_name: myapp-php

    restart: unless-stopped

    working_dir: /var/www/html

    volumes:

      - ./app:/var/www/html

      - web-static:/var/www/html/static

    environment:

      - APP_ENV=production

      - APP_DEBUG=false

      - DB_HOST=db

      - DB_PORT=3306

      - DB_DATABASE=${DB_DATABASE}

      - DB_USERNAME=${DB_USERNAME}

      - DB_PASSWORD=${DB_PASSWORD}

      - REDIS_HOST=redis

      - REDIS_PORT=6379

    env_file:

      - .env

    depends_on:

      db:

        condition: service_healthy

      redis:

        condition: service_started

    networks:

      - backend

    deploy:

      resources:

        limits:

          cpus: '2'

          memory: 1G

        reservations:

          cpus: '0.5'

          memory: 512M

  

  # MySQL база данных

  db:

    image: mysql:8.0

    container_name: myapp-mysql

    restart: unless-stopped

    ports:

      - "3306:3306"

    volumes:

      - db-data:/var/lib/mysql

      - ./mysql/init:/docker-entrypoint-initdb.d:ro

      - ./mysql/conf.d:/etc/mysql/conf.d:ro

    environment:

      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}

      - MYSQL_DATABASE=${DB_DATABASE}

      - MYSQL_USER=${DB_USERNAME}

      - MYSQL_PASSWORD=${DB_PASSWORD}

    command: --default-authentication-plugin=mysql_native_password

    networks:

      - backend

    healthcheck:

      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${DB_ROOT_PASSWORD}"]

      interval: 10s

      timeout: 5s

      retries: 5

      start_period: 30s

  

  # Redis кэш

  redis:

    image: redis:7-alpine

    container_name: myapp-redis

    restart: unless-stopped

    ports:

      - "6379:6379"

    volumes:

      - redis-data:/data

      - ./redis/redis.conf:/usr/local/etc/redis/redis.conf:ro

    command: redis-server /usr/local/etc/redis/redis.conf

    networks:

      - backend

    healthcheck:

      test: ["CMD", "redis-cli", "ping"]

      interval: 10s

      timeout: 3s

      retries: 5

  

  # Worker для очередей

  worker:

    build:

      context: ./app

      dockerfile: Dockerfile

    container_name: myapp-worker

    restart: unless-stopped

    command: php artisan queue:work --sleep=3 --tries=3

    volumes:

      - ./app:/var/www/html

    environment:

      - APP_ENV=production

      - DB_HOST=db

      - REDIS_HOST=redis

    env_file:

      - .env

    depends_on:

      - db

      - redis

    networks:

      - backend

    deploy:

      replicas: 2

  

  # Scheduler (cron)

  scheduler:

    build:

      context: ./app

      dockerfile: Dockerfile

    container_name: myapp-scheduler

    restart: unless-stopped

    command: php artisan schedule:work

    volumes:

      - ./app:/var/www/html

    environment:

      - APP_ENV=production

      - DB_HOST=db

      - REDIS_HOST=redis

    env_file:

      - .env

    depends_on:

      - db

      - redis

    networks:

      - backend

  

networks:

  frontend:

    driver: bridge

  backend:

    driver: bridge

    internal: true  # нет доступа к внешней сети

  

volumes:

  db-data:

    driver: local

  redis-data:

    driver: local

  web-static:

    driver: local

```

  

**Файл .env:**

```env

# Database

DB_ROOT_PASSWORD=supersecret

DB_DATABASE=myapp

DB_USERNAME=myapp_user

DB_PASSWORD=secret123

  

# Application

APP_KEY=base64:randomgeneratedkey

APP_URL=https://myapp.com

```

  

**### Основные команды Docker Compose**

  

**Управление жизненным циклом:**

```bash

# Запустить все сервисы

docker-compose up

  

# Запустить в фоне (detached mode)

docker-compose up -d

  

# Запустить конкретный сервис

docker-compose up nginx

  

# Пересобрать образы перед запуском

docker-compose up --build

  

# Пересоздать контейнеры

docker-compose up --force-recreate

  

# Остановить все сервисы

docker-compose stop

  

# Остановить и удалить контейнеры

docker-compose down

  

# Остановить и удалить контейнеры, сети, volumes

docker-compose down -v

  

# Остановить и удалить всё, включая образы

docker-compose down --rmi all -v

```

  

**Управление отдельными сервисами:**

```bash

# Запустить остановленный сервис

docker-compose start nginx

  

# Остановить сервис

docker-compose stop nginx

  

# Перезапустить сервис

docker-compose restart nginx

  

# Приостановить сервис

docker-compose pause nginx

  

# Возобновить сервис

docker-compose unpause nginx

  

# Удалить остановленный сервис

docker-compose rm nginx

```

  

**Сборка образов:**

```bash

# Собрать все образы

docker-compose build

  

# Собрать конкретный сервис

docker-compose build app

  

# Собрать без кэша

docker-compose build --no-cache

  

# Параллельная сборка

docker-compose build --parallel

```

  

**Просмотр информации:**

```bash

# Список запущенных контейнеров

docker-compose ps

  

# Список всех контейнеров (включая остановленные)

docker-compose ps -a

  

# Просмотр логов всех сервисов

docker-compose logs

  

# Следить за логами

docker-compose logs -f

  

# Логи конкретного сервиса

docker-compose logs -f nginx

  

# Последние N строк

docker-compose logs --tail=100 app

  

# Список образов

docker-compose images

  

# Использование ресурсов

docker-compose top

  

# События от сервисов

docker-compose events

```

  

**Выполнение команд:**

```bash

# Выполнить команду в запущенном контейнере

docker-compose exec app bash

docker-compose exec db mysql -u root -p

  

# Выполнить команду без TTY (для скриптов)

docker-compose exec -T app php artisan migrate

  

# Выполнить команду с созданием нового контейнера

docker-compose run app php artisan tinker

  

# Без зависимостей

docker-compose run --no-deps app npm test

  

# Удалить контейнер после выполнения

docker-compose run --rm app php artisan test

```

  

**Масштабирование:**

```bash

# Запустить несколько экземпляров сервиса

docker-compose up -d --scale worker=3

  

# Масштабировать без пересоздания

docker-compose scale worker=5

```

  

**Проверка конфигурации:**

```bash

# Валидация синтаксиса

docker-compose config

  

# Просмотр финальной конфигурации (с подстановкой переменных)

docker-compose config --services

  

# Список сервисов

docker-compose config --services

  

# Просмотр volumes

docker-compose config --volumes

```

  

**### Расширенные возможности Compose**

  

**1. Множественные файлы Compose**

```bash

# Базовый файл + override

docker-compose -f docker-compose.yml -f docker-compose.override.yml up

  

# Разработка

docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

  

# Продакшн

docker-compose -f docker-compose.yml -f docker-compose.prod.yml up

```

  

**docker-compose.yml:**

```yaml

version: '3.8'

services:

  app:

    image: myapp:latest

    environment:

      - APP_ENV=production

```

  

**docker-compose.dev.yml:**

```yaml

version: '3.8'

services:

  app:

    build: .

    volumes:

      - .:/app

    environment:

      - APP_ENV=development

      - APP_DEBUG=true

```

  

**2. Расширение конфигураций (extends)**

```yaml

# common-services.yml

version: '3.8'

services:

  base-app:

    image: node:18

    restart: unless-stopped

    networks:

      - app-network

  

# docker-compose.yml

version: '3.8'

services:

  web:

    extends:

      file: common-services.yml

      service: base-app

    ports:

      - "3000:3000"

```

  

**3. Якоря и алиасы YAML**

```yaml

version: '3.8'

  

x-common-variables: &common-env

  APP_ENV: production

  LOG_LEVEL: info

  

x-common-healthcheck: &common-healthcheck

  interval: 30s

  timeout: 10s

  retries: 3

  start_period: 40s

  

services:

  app1:

    image: myapp:latest

    environment:

      <<: *common-env

      SERVICE_NAME: app1

    healthcheck:

      <<: *common-healthcheck

      test: ["CMD", "curl", "-f", "http://localhost/health"]

  

  app2:

    image: myapp:latest

    environment:

      <<: *common-env

      SERVICE_NAME: app2

    healthcheck:

      <<: *common-healthcheck

      test: ["CMD", "curl", "-f", "http://localhost/health"]

```

  

**4. Profiles (группы сервисов)**

```yaml

version: '3.8'

  

services:

  web:

    image: nginx

    ports:

      - "80:80"

  

  db:

    image: postgres

    # Базовый сервис - запускается всегда

  

  adminer:

    image: adminer

    profiles: ["debug"]

    ports:

      - "8080:8080"

  

  test:

    image: myapp:test

    profiles: ["test"]

    command: npm test

```

  

```bash

# Запустить без профилей (только web и db)

docker-compose up

  

# Запустить с профилем debug

docker-compose --profile debug up

  

# Запустить с несколькими профилями

docker-compose --profile debug --profile test up

```

  

**5. Зависимости между сервисами**

```yaml

services:

  web:

    image: nginx

    depends_on:

      app:

        condition: service_healthy

      db:

        condition: service_started

  

  app:

    image: myapp

    depends_on:

      - db

    healthcheck:

      test: ["CMD", "curl", "-f", "http://localhost/health"]

      interval: 10s

  

  db:

    image: postgres

    healthcheck:

      test: ["CMD-SHELL", "pg_isready -U postgres"]

      interval: 10s

```

  

**### Best Practices для Docker Compose**

  

**1. Используйте .env файлы**

```yaml

# docker-compose.yml

services:

  db:

    environment:

      - MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD}

      - MYSQL_DATABASE=${DB_NAME}

```

  

**2. Не коммитьте секреты**

```.gitignore

.env

.env.local

docker-compose.override.yml

```

  

**3. Используйте named volumes**

```yaml

volumes:

  db-data:

    driver: local

  redis-data:

    driver: local

```

  

**4. Настройте restart policy**

```yaml

services:

  app:

    restart: unless-stopped  # или: no, always, on-failure

```

  

**5. Добавьте healthchecks**

```yaml

services:

  app:

    healthcheck:

      test: ["CMD", "curl", "-f", "http://localhost/health"]

      interval: 30s

      timeout: 10s

      retries: 3

      start_period: 40s

```

  

**6. Ограничивайте ресурсы**

```yaml

services:

  app:

    deploy:

      resources:

        limits:

          cpus: '2'

          memory: 2G

        reservations:

          cpus: '1'

          memory: 1G

```

  

**7. Используйте logging**

```yaml

services:

  app:

    logging:

      driver: "json-file"

      options:

        max-size: "10m"

        max-file: "3"

```

  

**8. Разделяйте сети**

```yaml

services:

  nginx:

    networks:

      - frontend

      - backend

  app:

    networks:

      - backend

  db:

    networks:

      - backend

  

networks:

  frontend:

  backend:

    internal: true  # нет доступа во внешний мир

```

  

**### Примеры реальных стеков**

  

**LEMP Stack (Linux, Nginx, MySQL, PHP):**

```yaml

version: '3.8'

  

services:

  nginx:

    image: nginx:alpine

    ports:

      - "80:80"

    volumes:

      - ./src:/var/www/html

      - ./nginx.conf:/etc/nginx/conf.d/default.conf

    depends_on:

      - php

  

  php:

    build:

      context: .

      dockerfile: Dockerfile.php

    volumes:

      - ./src:/var/www/html

    depends_on:

      - mysql

  

  mysql:

    image: mysql:8.0

    environment:

      MYSQL_ROOT_PASSWORD: root

      MYSQL_DATABASE: myapp

    volumes:

      - mysql-data:/var/lib/mysql

  

volumes:

  mysql-data:

```

  

**MERN Stack (MongoDB, Express, React, Node):**

```yaml

version: '3.8'

  

services:

  frontend:

    build: ./frontend

    ports:

      - "3000:3000"

    volumes:

      - ./frontend:/app

      - /app/node_modules

    environment:

      - REACT_APP_API_URL=http://localhost:5000

  

  backend:

    build: ./backend

    ports:

      - "5000:5000"

    volumes:

      - ./backend:/app

      - /app/node_modules

    environment:

      - MONGO_URI=mongodb://mongo:27017/myapp

    depends_on:

      - mongo

  

  mongo:

    image: mongo:6

    ports:

      - "27017:27017"

    volumes:

      - mongo-data:/data/db

  

volumes:

  mongo-data:

```

  

**WordPress Stack:**

```yaml

version: '3.8'

  

services:

  wordpress:

    image: wordpress:latest

    ports:

      - "8000:80"

    environment:

      WORDPRESS_DB_HOST: db

      WORDPRESS_DB_USER: wordpress

      WORDPRESS_DB_PASSWORD: wordpress

      WORDPRESS_DB_NAME: wordpress

    volumes:

      - wordpress-data:/var/www/html

    depends_on:

      - db

  

  db:

    image: mysql:8.0

    environment:

      MYSQL_ROOT_PASSWORD: rootpassword

      MYSQL_DATABASE: wordpress

      MYSQL_USER: wordpress

      MYSQL_PASSWORD: wordpress

    volumes:

      - db-data:/var/lib/mysql

  

  phpmyadmin:

    image: phpmyadmin:latest

    ports:

      - "8080:80"

    environment:

      PMA_HOST: db

      PMA_USER: wordpress

      PMA_PASSWORD: wordpress

    depends_on:

      - db

  

volumes:

  wordpress-data:

  db-data:

```

  

**### Troubleshooting и отладка**

  

**Проблемы с сетью:**

```bash

# Проверить сети

docker network ls

docker network inspect myapp_default

  

# Пересоздать сети

docker-compose down

docker-compose up

```

  

**Проблемы с volumes:**

```bash

# Проверить volumes

docker volume ls

docker volume inspect myapp_db-data

  

# Очистить volumes

docker-compose down -v

```

  

**Проблемы с портами:**

```bash

# Проверить занятые порты

lsof -i :80

netstat -tuln | grep 80

  

# Изменить порт в docker-compose.yml

ports:

  - "8080:80"  # вместо "80:80"

```

  

**Проблемы с зависимостями:**

```bash

# Запустить сервисы по одному

docker-compose up db

docker-compose up app

docker-compose up nginx

```

  

**Проверка конфигурации:**

```bash

# Валидация

docker-compose config

  

# Просмотр переменных

docker-compose config --resolve-image-digests

```

  

**### Когда использовать Docker Compose**