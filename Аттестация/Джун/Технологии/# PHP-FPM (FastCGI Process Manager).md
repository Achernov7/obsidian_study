
**## Содержание**

1. [Что такое PHP-FPM](#что-такое-php-fpm)

2. [Как работает PHP-FPM](#как-работает-php-fpm)

3. [Базовые настройки](#базовые-настройки)

4. [Как запускается на сервере](#как-запускается-на-сервере)

5. [Мониторинг и отладка](#мониторинг-и-отладка)

6. [Оптимизация](#оптимизация)

7. [Шпаргалка](#шпаргалка)

  

---

  

**## Что такое PHP-FPM**

  

**PHP-FPM (FastCGI Process Manager)** - это альтернативная реализация PHP FastCGI с дополнительными возможностями для высоконагруженных сайтов.

  

### Простыми словами:

  

> PHP-FPM - это "менеджер процессов", который запускает PHP-скрипты и управляет рабочими процессами (workers).

  

---

  

**### Зачем нужен PHP-FPM:**

  

**Без PHP-FPM (старый способ - mod_php):**

  

```

Браузер → Apache (с встроенным PHP) → PHP скрипт

  

Проблемы:

❌ Apache загружает PHP в каждый свой процесс (даже для статики)

❌ Потребляет много памяти

❌ Сложно масштабировать

❌ Нельзя использовать разные версии PHP

```

  

**С PHP-FPM:**

  

```

Браузер → Nginx/Apache → PHP-FPM → PHP скрипт

  

Преимущества:

✅ PHP работает отдельно от веб-сервера

✅ Nginx может отдавать статику сам (быстрее)

✅ Меньше потребление памяти

✅ Легко масштабировать

✅ Можно запустить несколько версий PHP одновременно

✅ Лучший контроль над процессами

```

  

---

  

**### Аналогия из жизни:**

  

**Без PHP-FPM (mod_php):**

```

🏪 Магазин с продавцами

  

Каждый продавец (Apache процесс) = универсальный:

- Продает товары

- Принимает оплату

- Выносит мусор

- Готовит кофе

  

❌ Даже если клиент хочет просто взять товар,

   продавец занят другими делами

```

  

**С PHP-FPM:**

```

🏪 Магазин + отдельная кофейня

  

Продавцы (Nginx):

- Быстро отдают товары (статику)

- Передают заказы на кофе (PHP) в кофейню

  

Бариста (PHP-FPM):

- Специализируются ТОЛЬКО на кофе (PHP)

- Можно нанять больше/меньше бариста

- Отдельная очередь для кофе

  

✅ Клиенты за товарами не ждут пока готовится кофе

✅ Можно масштабировать отдельно

```

  

---

  

**### Архитектура PHP-FPM:**

  

```

┌─────────────────────────────────────────────────────┐

│  Клиент (Браузер)                                   │

└──────────────────┬──────────────────────────────────┘

                   │ HTTP запрос

                   ▼

┌─────────────────────────────────────────────────────┐

│  Веб-сервер (Nginx / Apache)                        │

│  ┌─────────────────┐  ┌──────────────────────────┐ │

│  │ Статика         │  │ PHP запросы              │ │

│  │ (CSS, JS, IMG)  │  │ (*.php)                  │ │

│  └─────────────────┘  └──────────────┬───────────┘ │

└────────────────────────────────────────┼────────────┘

                                         │ FastCGI

                                         ▼

┌─────────────────────────────────────────────────────┐

│  PHP-FPM (Master процесс)                           │

│  ┌──────────────────────────────────────────────┐  │

│  │  Pool: www                                   │  │

│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐        │  │

│  │  │ Worker 1│ │ Worker 2│ │ Worker 3│  ...   │  │

│  │  │ (PHP)   │ │ (PHP)   │ │ (PHP)   │        │  │

│  │  └─────────┘ └─────────┘ └─────────┘        │  │

│  └──────────────────────────────────────────────┘  │

│  ┌──────────────────────────────────────────────┐  │

│  │  Pool: api (другая версия PHP)              │  │

│  │  ┌─────────┐ ┌─────────┐                     │  │

│  │  │ Worker 1│ │ Worker 2│  ...                │  │

│  │  └─────────┘ └─────────┘                     │  │

│  └──────────────────────────────────────────────┘  │

└─────────────────────────────────────────────────────┘

  

Master процесс:

- Управляет пулами (pools)

- Создает/убивает worker процессы

- Обрабатывает сигналы (reload, restart)

  

Worker процесс:

- Выполняет PHP код

- Обрабатывает один запрос за раз

- Автоматически перезапускается после N запросов

```

  

---

  

**## Как работает PHP-FPM**

  

**### Процесс обработки запроса:**

  

```

1. Браузер → GET /index.php

   ↓

2. Nginx получает запрос

   ↓

3. Nginx определяет: это PHP файл

   ↓

4. Nginx отправляет запрос в PHP-FPM через FastCGI

   (обычно через Unix socket или TCP)

   ↓

5. PHP-FPM Master выбирает свободный Worker

   ↓

6. Worker выполняет PHP скрипт

   ↓

7. Worker возвращает результат в Nginx

   ↓

8. Nginx отправляет ответ браузеру

   ↓

9. Worker освобождается для следующего запроса

```

  

---

  

**### Пулы (Pools):**

  

**Пул** - это группа worker процессов с общими настройками.

  

```

Зачем нужны пулы:

  

✅ Разные сайты на одном сервере:

   - pool "site1" → /var/www/site1

   - pool "site2" → /var/www/site2

  

✅ Разные версии PHP:

   - pool "legacy" → PHP 7.4

   - pool "modern" → PHP 8.2

  

✅ Разные лимиты:

   - pool "frontend" → 50 workers (много легких запросов)

   - pool "backend" → 10 workers (мало тяжелых запросов)

  

✅ Изоляция:

   - Разные пользователи

   - Разные лимиты памяти

   - Если один пул сломался, другие работают

```

  

---

  

**## Базовые настройки**

  

**### Основные файлы конфигурации:**

  

```bash

# Главный конфигурационный файл

/etc/php/8.2/fpm/php-fpm.conf

  

# Настройки пулов (пул по умолчанию)

/etc/php/8.2/fpm/pool.d/www.conf

  

# Конфигурация PHP (php.ini)

/etc/php/8.2/fpm/php.ini

```

  

---

  

**### Главный файл php-fpm.conf**

  

```ini

; /etc/php/8.2/fpm/php-fpm.conf

  

; PID файл процесса

pid = /run/php/php8.2-fpm.pid

  

; Уровень логирования (alert, error, warning, notice, debug)

error_log = /var/log/php8.2-fpm.log

log_level = notice

  

; Аварийный лог (для серьезных ошибок)

emergency_restart_threshold = 10

emergency_restart_interval = 1m

  

; Лимит времени обработки дочерних процессов

process_control_timeout = 10s

  

; Максимальное количество дочерних процессов (глобально)

; process.max = 128

  

; Включить директорию с пулами

include=/etc/php/8.2/fpm/pool.d/*.conf

```

  

---

  

**### Настройки пула (pool.d/www.conf)**

  

Это ОСНОВНОЙ файл с настройками:

  

```ini

; /etc/php/8.2/fpm/pool.d/www.conf

  

; ============================================

; ОСНОВНЫЕ НАСТРОЙКИ

; ============================================

  

; Имя пула

[www]

  

; Пользователь и группа, от имени которых запускаются процессы

user = www-data

group = www-data

  

; Способ подключения: Unix socket (быстрее) или TCP

listen = /run/php/php8.2-fpm.sock

; или

; listen = 127.0.0.1:9000

  

; Права доступа к socket файлу

listen.owner = www-data

listen.group = www-data

listen.mode = 0660

  

; Разрешенные клиенты (для TCP)

; listen.allowed_clients = 127.0.0.1

  

; ============================================

; УПРАВЛЕНИЕ ПРОЦЕССАМИ

; ============================================

  

; Менеджер процессов:

; - static:  фиксированное количество workers

; - dynamic: динамическое количество (рекомендуется)

; - ondemand: создаются по требованию

pm = dynamic

  

; Максимальное количество дочерних процессов

pm.max_children = 50

  

; Количество процессов при старте (только для dynamic)

pm.start_servers = 5

  

; Минимальное количество свободных процессов

pm.min_spare_servers = 5

  

; Максимальное количество свободных процессов

pm.max_spare_servers = 35

  

; Максимальное количество запросов, которое обработает worker

; перед перезапуском (защита от утечек памяти)

pm.max_requests = 500

  

; ============================================

; ТАЙМАУТЫ

; ============================================

  

; Максимальное время выполнения скрипта (секунды)

request_terminate_timeout = 30s

  

; Таймаут ожидания запроса (для keep-alive)

request_slowlog_timeout = 5s

  

; Лог медленных запросов

slowlog = /var/log/php8.2-fpm-slow.log

  

; ============================================

; ЛОГИРОВАНИЕ

; ============================================

  

; Логировать stdout/stderr от PHP скриптов

catch_workers_output = yes

  

; Декорировать логи (добавлять worker ID)

decorate_workers_output = no

  

; ============================================

; ЛИМИТЫ РЕСУРСОВ

; ============================================

  

; Максимальное время CPU (секунды)

; rlimit_cpu = unlimited

  

; Максимальный размер core dump

; rlimit_core = unlimited

  

; Максимальное количество открытых файлов

; rlimit_files = 1024

  

; ============================================

; ПЕРЕМЕННЫЕ ОКРУЖЕНИЯ

; ============================================

  

; Передать переменные окружения в PHP

env[HOSTNAME] = $HOSTNAME

env[PATH] = /usr/local/bin:/usr/bin:/bin

env[TMP] = /tmp

env[TMPDIR] = /tmp

env[TEMP] = /tmp

  

; ============================================

; PHP НАСТРОЙКИ (переопределение php.ini)

; ============================================

  

; Лимит памяти

php_admin_value[memory_limit] = 256M

  

; Максимальное время выполнения

php_admin_value[max_execution_time] = 30

  

; Размер POST данных

php_admin_value[post_max_size] = 100M

  

; Размер загружаемых файлов

php_admin_value[upload_max_filesize] = 100M

  

; Отображение ошибок (для production = Off)

php_flag[display_errors] = Off

  

; Логирование ошибок

php_flag[log_errors] = On

php_value[error_log] = /var/log/php-errors.log

  

; Временная папка для сессий

php_value[session.save_path] = /var/lib/php/sessions

```

  

---

  

**### Режимы управления процессами (pm)**

  

#### 1. static - Фиксированное количество

  

```ini

pm = static

pm.max_children = 20

  

# Всегда 20 worker процессов

# Не зависит от нагрузки

```

  

**Плюсы:**

- ✅ Предсказуемое потребление памяти

- ✅ Нет задержек на создание процессов

  

**Минусы:**

- ❌ Расход памяти даже без нагрузки

- ❌ Нельзя обработать больше запросов, чем workers

  

**Когда использовать:**

- Стабильная высокая нагрузка

- Достаточно памяти

- Production серверы с предсказуемым трафиком

  

---

  

#### 2. dynamic - Динамическое количество (РЕКОМЕНДУЕТСЯ)

  

```ini

pm = dynamic

pm.max_children = 50         # Максимум

pm.start_servers = 5         # При старте

pm.min_spare_servers = 5     # Минимум свободных

pm.max_spare_servers = 35    # Максимум свободных

  

# PHP-FPM автоматически создает/убивает процессы

# в зависимости от нагрузки

```

  

**Как работает:**

```

Запуск:

- Создается 5 workers (start_servers)

  

Нагрузка растет:

- Если свободных < 5, создаются новые

- До максимума 50 (max_children)

  

Нагрузка падает:

- Если свободных > 35, лишние убиваются

- Минимум остается 5 (min_spare_servers)

```

  

**Плюсы:**

- ✅ Экономия памяти при низкой нагрузке

- ✅ Масштабирование при высокой нагрузке

- ✅ Универсальный режим

  

**Минусы:**

- ⚠️ Небольшая задержка при создании новых процессов

  

**Когда использовать:**

- Большинство случаев

- Переменная нагрузка

- Ограниченная память

  

---

  

#### 3. ondemand - По требованию

  

```ini

pm = ondemand

pm.max_children = 50

pm.process_idle_timeout = 10s

  

# Процессы создаются только при запросах

# Убиваются через 10s после простоя

```

  

**Плюсы:**

- ✅ Минимальное потребление памяти

- ✅ Идеально для dev/staging серверов

  

**Минусы:**

- ❌ Задержка на первом запросе (создание процесса)

- ❌ Не подходит для production

  

**Когда использовать:**

- Локальная разработка

- Множество сайтов на одном сервере (shared hosting)

- Редко используемые сайты

  

---

  

### Расчет pm.max_children

  

**Формула:**

  

```

pm.max_children = Доступная_память / Память_на_процесс

  

Где:

- Доступная_память = RAM - память_для_системы - память_для_других_сервисов

- Память_на_процесс ≈ 50-100 MB (зависит от приложения)

```

  

**Пример:**

  

```bash

Сервер: 4 GB RAM

- Система: 512 MB

- MySQL: 1 GB

- Nginx: 100 MB

- Прочее: 388 MB

= Для PHP-FPM: ~2 GB

  

Средний PHP процесс: 80 MB

  

pm.max_children = 2000 MB / 80 MB = 25 процессов

```

  

**Проверить реальное потребление:**

  

```bash

# Память одного PHP-FPM процесса

ps aux | grep php-fpm | awk '{sum+=$6} END {print sum/NR/1024 " MB"}'

  

# Общая память всех PHP-FPM процессов

ps aux | grep php-fpm | awk '{sum+=$6} END {print sum/1024 " MB"}'

  

# Количество процессов

ps aux | grep php-fpm | grep -c pool

```

  

---

  

**### Настройки для разных типов нагрузки**

  

#### Легкие запросы (много простых страниц):

  

```ini

pm = dynamic

pm.max_children = 100

pm.start_servers = 20

pm.min_spare_servers = 10

pm.max_spare_servers = 50

pm.max_requests = 1000

request_terminate_timeout = 10s

```

  

#### Тяжелые запросы (API, обработка данных):

  

```ini

pm = dynamic

pm.max_children = 20

pm.start_servers = 5

pm.min_spare_servers = 3

pm.max_spare_servers = 10

pm.max_requests = 200

request_terminate_timeout = 60s

php_admin_value[memory_limit] = 512M

```

  

#### Dev/Staging:

  

```ini

pm = ondemand

pm.max_children = 10

pm.process_idle_timeout = 10s

request_terminate_timeout = 300s

php_flag[display_errors] = On

```

  

---

  

**## Как запускается на сервере**

  

**### Установка PHP-FPM**

  

#### Ubuntu/Debian:

  

```bash

# Обновить списки пакетов

sudo apt update

  

# Установить PHP-FPM

sudo apt install php8.2-fpm

  

# Установить дополнительные модули

sudo apt install php8.2-mysql php8.2-curl php8.2-mbstring \

                 php8.2-xml php8.2-zip php8.2-gd php8.2-intl

  

# Проверить установку

php-fpm8.2 -v

```

  

#### CentOS/RHEL:

  

```bash

# Установить EPEL и Remi репозитории

sudo yum install epel-release

sudo yum install http://rpms.remirepo.net/enterprise/remi-release-8.rpm

  

# Включить PHP 8.2

sudo yum module enable php:remi-8.2

  

# Установить PHP-FPM

sudo yum install php-fpm php-mysqlnd php-curl php-mbstring \

                 php-xml php-zip php-gd

```

  

---

  

**### Управление службой**

  

```bash

# Запустить PHP-FPM

sudo systemctl start php8.2-fpm

  

# Остановить

sudo systemctl stop php8.2-fpm

  

# Перезапустить (полный restart)

sudo systemctl restart php8.2-fpm

  

# Reload конфигурации (без прерывания запросов)

sudo systemctl reload php8.2-fpm

  

# Автозапуск при старте системы

sudo systemctl enable php8.2-fpm

  

# Статус

sudo systemctl status php8.2-fpm

  

# Логи

sudo journalctl -u php8.2-fpm -f

```

  

---

  

**### Проверка конфигурации**

  

```bash

# Проверить синтаксис конфигурации

sudo php-fpm8.2 -t

  

# Вывод:

# [26-Oct-2024 16:00:00] NOTICE: configuration file /etc/php/8.2/fpm/php-fpm.conf test is successful

  

# Показать все настройки

sudo php-fpm8.2 -tt

  

# Показать модули

php-fpm8.2 -m

```

  

---

  

**### Настройка Nginx для работы с PHP-FPM**

  

```nginx

# /etc/nginx/sites-available/example.com

  

server {

    listen 80;

    server_name example.com;

    root /var/www/example.com;

    index index.php index.html;

    # Логи

    access_log /var/log/nginx/example.com-access.log;

    error_log /var/log/nginx/example.com-error.log;

    # PHP файлы обрабатываются через PHP-FPM

    location ~ \.php$ {

        # Защита от атак через Nginx

        try_files $uri =404;

        # Передать запрос в PHP-FPM через Unix socket

        fastcgi_pass unix:/run/php/php8.2-fpm.sock;

        # Или через TCP (если используется listen = 127.0.0.1:9000)

        # fastcgi_pass 127.0.0.1:9000;

        # Индексный файл

        fastcgi_index index.php;

        # Параметры FastCGI

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

        # Таймауты

        fastcgi_read_timeout 300;

        fastcgi_send_timeout 300;

        # Буферы (для больших ответов)

        fastcgi_buffers 16 16k;

        fastcgi_buffer_size 32k;

    }

    # Статические файлы (обрабатываются Nginx напрямую)

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf)$ {

        expires 30d;

        access_log off;

    }

    # Запретить доступ к скрытым файлам

    location ~ /\. {

        deny all;

    }

}

```

  

```bash

# Активировать конфигурацию

sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

  

# Проверить конфигурацию Nginx

sudo nginx -t

  

# Перезагрузить Nginx

sudo systemctl reload nginx

```

  

---

  

**### Несколько пулов для разных сайтов**

  

#### Создать пул для site1:

  

```bash

sudo nano /etc/php/8.2/fpm/pool.d/site1.conf

```

  

```ini

[site1]

user = site1user

group = site1user

listen = /run/php/php8.2-fpm-site1.sock

listen.owner = www-data

listen.group = www-data

  

pm = dynamic

pm.max_children = 20

pm.start_servers = 5

pm.min_spare_servers = 3

pm.max_spare_servers = 10

  

php_admin_value[memory_limit] = 128M

php_admin_value[upload_max_filesize] = 50M

```

  

#### Создать пул для site2:

  

```bash

sudo nano /etc/php/8.2/fpm/pool.d/site2.conf

```

  

```ini

[site2]

user = site2user

group = site2user

listen = /run/php/php8.2-fpm-site2.sock

listen.owner = www-data

listen.group = www-data

  

pm = dynamic

pm.max_children = 30

pm.start_servers = 10

pm.min_spare_servers = 5

pm.max_spare_servers = 15

  

php_admin_value[memory_limit] = 256M

php_admin_value[upload_max_filesize] = 100M

```

  

#### Nginx конфигурация для site1:

  

```nginx

server {

    server_name site1.com;

    root /var/www/site1;

    location ~ \.php$ {

        fastcgi_pass unix:/run/php/php8.2-fpm-site1.sock;

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    }

}

```

  

#### Nginx конфигурация для site2:

  

```nginx

server {

    server_name site2.com;

    root /var/www/site2;

    location ~ \.php$ {

        fastcgi_pass unix:/run/php/php8.2-fpm-site2.sock;

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    }

}

```

  

```bash

# Перезагрузить PHP-FPM

sudo systemctl reload php8.2-fpm

  

# Проверить что пулы запустились

ps aux | grep php-fpm

```

  

---

  

**### Автозапуск и systemd**

  

```bash

# Файл службы (обычно уже создан при установке)

cat /lib/systemd/system/php8.2-fpm.service

  

# Содержимое:

[Unit]

Description=The PHP 8.2 FastCGI Process Manager

After=network.target

  

[Service]

Type=notify

PIDFile=/run/php/php8.2-fpm.pid

ExecStart=/usr/sbin/php-fpm8.2 --nodaemonize --fpm-config /etc/php/8.2/fpm/php-fpm.conf

ExecReload=/bin/kill -USR2 $MAINPID

Restart=on-failure

  

[Install]

WantedBy=multi-user.target

  

# Перечитать конфигурацию systemd (если изменили)

sudo systemctl daemon-reload

```

  

---

  

**## Мониторинг и отладка**

  

**### Статус PHP-FPM**

  

```bash

# Текущий статус службы

sudo systemctl status php8.2-fpm

  

# Вывод:

● php8.2-fpm.service - The PHP 8.2 FastCGI Process Manager

     Loaded: loaded (/lib/systemd/system/php8.2-fpm.service; enabled)

     Active: active (running) since Sat 2024-10-26 15:00:00 UTC; 2h ago

   Main PID: 1234 (php-fpm8.2)

     Status: "Processes active: 5, idle: 10, Requests: 12345"

      Tasks: 16 (limit: 4915)

     Memory: 1.2G

        CPU: 5min 23s

```

  

---

  

**### Список процессов**

  

```bash

# Все PHP-FPM процессы

ps aux | grep php-fpm

  

# Вывод:

root     1234  0.0  0.1  php-fpm: master process

www-data 1235  0.1  2.5  php-fpm: pool www

www-data 1236  0.0  2.3  php-fpm: pool www

www-data 1237  0.2  2.7  php-fpm: pool www

  

# Количество процессов

ps aux | grep php-fpm | grep -c pool

  

# Использование памяти

ps aux | grep php-fpm | awk '{sum+=$6} END {print sum/1024 " MB"}'

```

  

---

  

**### Встроенная страница статуса**

  

Включить в конфигурации пула:

  

```ini

; /etc/php/8.2/fpm/pool.d/www.conf

  

; Страница статуса

pm.status_path = /fpm-status

  

; Ping страница

ping.path = /fpm-ping

ping.response = pong

```

  

Настроить Nginx:

  

```nginx

server {

    listen 127.0.0.1:80;

    location /fpm-status {

        access_log off;

        allow 127.0.0.1;

        deny all;

        fastcgi_pass unix:/run/php/php8.2-fpm.sock;

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    }

    location /fpm-ping {

        access_log off;

        allow 127.0.0.1;

        deny all;

        fastcgi_pass unix:/run/php/php8.2-fpm.sock;

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

    }

}

```

  

Проверить:

  

```bash

# Статус (текст)

curl http://127.0.0.1/fpm-status

  

# Вывод:

pool:                 www

process manager:      dynamic

start time:           26/Oct/2024:15:00:00 +0000

start since:          7200

accepted conn:        12345

listen queue:         0

max listen queue:     5

listen queue len:     128

idle processes:       10

active processes:     5

total processes:      15

max active processes: 20

max children reached: 0

slow requests:        3

  

# Статус (JSON)

curl http://127.0.0.1/fpm-status?json

  

# Статус (XML)

curl http://127.0.0.1/fpm-status?xml

  

# Полный статус (все процессы)

curl http://127.0.0.1/fpm-status?full

  

# Ping

curl http://127.0.0.1/fpm-ping

# pong

```

  

---

  

**### Логи**

  

```bash

# Основной лог PHP-FPM

sudo tail -f /var/log/php8.2-fpm.log

  

# Лог медленных запросов

sudo tail -f /var/log/php8.2-fpm-slow.log

  

# Лог ошибок PHP

sudo tail -f /var/log/php-errors.log

  

# Через journalctl

sudo journalctl -u php8.2-fpm -f

  

# Последние 100 строк

sudo journalctl -u php8.2-fpm -n 100

```

  

---

  

**### Отладка проблем**

  

#### Проблема: 502 Bad Gateway

  

```bash

# Проверить что PHP-FPM запущен

sudo systemctl status php8.2-fpm

  

# Проверить socket файл

ls -la /run/php/php8.2-fpm.sock

  

# Проверить права доступа

# Nginx должен иметь права на чтение socket

  

# Проверить логи

sudo tail -f /var/log/nginx/error.log

sudo tail -f /var/log/php8.2-fpm.log

```

  

#### Проблема: Медленная работа

  

```bash

# Проверить количество активных процессов

curl http://127.0.0.1/fpm-status

  

# Если active processes = max_children:

# Нужно увеличить pm.max_children

  

# Проверить медленные запросы

sudo tail -f /var/log/php8.2-fpm-slow.log

  

# Проверить использование памяти

free -h

ps aux | grep php-fpm | awk '{sum+=$6} END {print sum/1024 " MB"}'

```

  

#### Проблема: Утечки памяти

  

```bash

# Уменьшить pm.max_requests (перезапуск после N запросов)

pm.max_requests = 100

  

# Мониторить память

watch -n 1 'ps aux | grep php-fpm | awk "{sum+=\$6} END {print sum/1024 \" MB\"}"'

```

  

---

  

**## Оптимизация**

  

**### Checklist оптимизации:**

  

```ini

# 1. Правильно рассчитать pm.max_children

pm.max_children = Доступная_RAM / Память_процесса

  

# 2. Использовать Unix socket вместо TCP

listen = /run/php/php8.2-fpm.sock

# Быстрее на ~10-15%

  

# 3. Увеличить лимиты для production

pm.max_requests = 500

request_terminate_timeout = 30s

  

# 4. Включить OPcache (в php.ini)

opcache.enable = 1

opcache.memory_consumption = 128

opcache.max_accelerated_files = 10000

opcache.validate_timestamps = 0  # Production

  

# 5. Отключить ненужные модули

; xdebug.mode = off  # Production!

  

# 6. Настроить буферы в Nginx

fastcgi_buffers 16 16k;

fastcgi_buffer_size 32k;

  

# 7. Мониторить и логировать

slowlog = /var/log/php-fpm-slow.log

request_slowlog_timeout = 5s

  

# 8. Использовать разные пулы

# Для разных сайтов/нагрузки

```

  

---

  

**## Шпаргалка**

  

**### Основные команды:**

  

```bash

# Установка

sudo apt install php8.2-fpm

  

# Управление

sudo systemctl start php8.2-fpm

sudo systemctl stop php8.2-fpm

sudo systemctl restart php8.2-fpm

sudo systemctl reload php8.2-fpm

sudo systemctl status php8.2-fpm

  

# Проверка конфигурации

sudo php-fpm8.2 -t

  

# Логи

sudo tail -f /var/log/php8.2-fpm.log

sudo journalctl -u php8.2-fpm -f

  

# Процессы

ps aux | grep php-fpm

  

# Статус

curl http://127.0.0.1/fpm-status

```

  

---

  

**### Основные файлы:**

  

```

Конфигурация:

/etc/php/8.2/fpm/php-fpm.conf       - главный файл

/etc/php/8.2/fpm/pool.d/www.conf   - настройки пула

/etc/php/8.2/fpm/php.ini            - настройки PHP

  

Runtime:

/run/php/php8.2-fpm.sock            - Unix socket

/run/php/php8.2-fpm.pid             - PID файл

  

Логи:

/var/log/php8.2-fpm.log             - основной лог

/var/log/php8.2-fpm-slow.log        - медленные запросы

```

  

---

  

**### Базовая конфигурация пула:**

  

```ini

[www]

user = www-data

group = www-data

listen = /run/php/php8.2-fpm.sock

listen.owner = www-data

listen.group = www-data

  

pm = dynamic

pm.max_children = 50

pm.start_servers = 5

pm.min_spare_servers = 5

pm.max_spare_servers = 35

pm.max_requests = 500

  

request_terminate_timeout = 30s

slowlog = /var/log/php8.2-fpm-slow.log

request_slowlog_timeout = 5s

  

php_admin_value[memory_limit] = 256M

php_admin_value[upload_max_filesize] = 100M

```

  

---

  

**### Nginx + PHP-FPM:**

  

```nginx

location ~ \.php$ {

    try_files $uri =404;

    fastcgi_pass unix:/run/php/php8.2-fpm.sock;

    fastcgi_index index.php;

    include fastcgi_params;

    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

}

```

  

---

  

**### Режимы pm:**

  

```

static   - фиксированное количество (для высокой нагрузки)

dynamic  - автоматическое управление (рекомендуется)

ondemand - по требованию (для dev/staging)

```

  

---

  

**### Формула расчета:**