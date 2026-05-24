
**## Содержание**

  

1. [Что такое LAMP и LEMP](#что-такое-lamp-и-lemp)

2. [Различия между LAMP и LEMP](#различия-между-lamp-и-lemp)

3. [Развертывание на Docker](#развертывание-на-docker)

4. [Установка на различные ОС](#установка-на-различные-ос)

5. [Способы запуска PHP](#способы-запуска-php)

6. [Пользователи и права доступа](#пользователи-и-права-доступа)

7. [Оптимизация и безопасность](#оптимизация-и-безопасность)

  

---

  

**## Что такое LAMP и LEMP**

  

**### LAMP Stack**

  

**LAMP** — это акроним из первых букв компонентов стека:

  

- **L** — Linux (операционная система)

- **A** — Apache (веб-сервер)

- **M** — MySQL/MariaDB (система управления базами данных)

- **P** — PHP (язык программирования)

  

**### LEMP Stack**

  

**LEMP** — альтернативный стек:

  

- **L** — Linux (операционная система)

- **E** — Nginx (Engine-X, веб-сервер)

- **M** — MySQL/MariaDB (СУБД)

- **P** — PHP (язык программирования)

  

**### Зачем нужны эти стеки?**

  

```

Клиент (браузер)

       ↓

Веб-сервер (Apache/Nginx)

       ↓

PHP Интерпретатор

       ↓

База данных (MySQL/MariaDB)

       ↓

Файловая система

```

  

**Основные задачи:**

  

1. **Обработка HTTP-запросов** — веб-сервер принимает запросы от клиентов

2. **Выполнение PHP-кода** — интерпретатор обрабатывает серверную логику

3. **Работа с данными** — СУБД хранит и обрабатывает данные

4. **Отдача статики** — раздача CSS, JS, изображений

  

---

  

**## Различия между LAMP и LEMP**

  

**### Архитектурные различия**

  

| Характеристика | Apache (LAMP) | Nginx (LEMP) |

|---------------|---------------|--------------|

| **Архитектура** | Process-driven (многопроцессная) | Event-driven (событийная) |

| **Обработка запросов** | Один процесс/поток на соединение | Асинхронная обработка |

| **Потребление памяти** | Выше (каждый процесс ~5-10 MB) | Ниже (меньше процессов) |

| **Обработка статики** | Хорошо | Отлично (быстрее) |

| **Обработка PHP** | mod_php (встроенный модуль) | PHP-FPM (отдельный процесс) |

| **Конфигурация** | .htaccess (распределенная) | Централизованная |

| **Гибкость настройки** | Очень высокая | Хорошая |

| **Производительность** | Хорошая при малой нагрузке | Отличная при высокой нагрузке |

  

**### Процесс обработки запросов**

  

**Apache (LAMP):**

  

```

Запрос → Apache

           ↓

    mod_php (встроенный модуль)

           ↓

    Выполнение PHP-кода

           ↓

    Ответ клиенту

```

  

**Nginx (LEMP):**

  

```

Запрос → Nginx

           ↓

    Проксирование → PHP-FPM (отдельный процесс)

                       ↓

                  Выполнение PHP-кода

                       ↓

    Ответ ← Nginx ← PHP-FPM

           ↓

    Ответ клиенту

```

  

**### Когда выбрать Apache?**

  

✅ **Преимущества:**

- Простота настройки для новичков

- Поддержка .htaccess (настройки на уровне директорий)

- Богатая экосистема модулей

- Отличная документация

- Встроенная поддержка PHP через mod_php

  

**Примеры использования:**

- Shared-хостинг (нужен .htaccess)

- Проекты, требующие специфичных Apache-модулей

- Legacy-приложения

- Небольшие проекты с малой нагрузкой

  

**### Когда выбрать Nginx?**

  

✅ **Преимущества:**

- Высокая производительность при большом количестве соединений

- Меньшее потребление ресурсов

- Отличная работа в роли reverse proxy

- Быстрая отдача статических файлов

- Масштабируемость

  

**Примеры использования:**

- Высоконагруженные проекты

- Микросервисная архитектура

- API-серверы

- Статические сайты

- Load balancer

  

**### Производительность: сравнение**

  

**Тест: 10,000 одновременных соединений**

  

```

Apache (mod_php):

- Память: ~800-1200 MB

- CPU: 60-80%

- Запросов/сек: 500-800

  

Nginx + PHP-FPM:

- Память: ~400-600 MB

- CPU: 30-50%

- Запросов/сек: 1200-2000

```

  

**Тест: Статические файлы (10,000 запросов)**

  

```

Apache:

- Время: 15-20 секунд

- Память: ~500 MB

  

Nginx:

- Время: 3-5 секунд

- Память: ~200 MB

```

  

---

  

**## Развертывание на Docker**

  

**### Вариант 1: Docker Compose с Apache (LAMP)**

  

**Структура проекта:**

  

```

project/

├── docker-compose.yml

├── Dockerfile

├── php.ini

├── apache/

│   └── default.conf

└── www/

    └── index.php

```

  

**docker-compose.yml (LAMP):**

  

```yaml

version: '3.8'

  

services:

  # Веб-сервер Apache + PHP

  web:

    build:

      context: .

      dockerfile: Dockerfile

    container_name: lamp_web

    ports:

      - "8080:80"

    volumes:

      - ./www:/var/www/html

      - ./php.ini:/usr/local/etc/php/php.ini

      - ./apache/default.conf:/etc/apache2/sites-available/000-default.conf

    environment:

      - APACHE_RUN_USER=www-data

      - APACHE_RUN_GROUP=www-data

    depends_on:

      - db

    networks:

      - lamp_network

  

  # База данных MySQL

  db:

    image: mysql:8.0

    container_name: lamp_mysql

    restart: always

    ports:

      - "3306:3306"

    environment:

      MYSQL_ROOT_PASSWORD: rootpassword

      MYSQL_DATABASE: myapp

      MYSQL_USER: appuser

      MYSQL_PASSWORD: apppassword

    volumes:

      - db_data:/var/lib/mysql

      - ./mysql/my.cnf:/etc/mysql/conf.d/my.cnf

    networks:

      - lamp_network

  

  # phpMyAdmin (опционально)

  phpmyadmin:

    image: phpmyadmin:latest

    container_name: lamp_phpmyadmin

    ports:

      - "8081:80"

    environment:

      PMA_HOST: db

      PMA_PORT: 3306

      PMA_USER: root

      PMA_PASSWORD: rootpassword

    depends_on:

      - db

    networks:

      - lamp_network

  

volumes:

  db_data:

  

networks:

  lamp_network:

    driver: bridge

```

  

**Dockerfile (Apache + PHP):**

  

```dockerfile

FROM php:8.2-apache

  

# Установка системных зависимостей

RUN apt-get update && apt-get install -y \

    git \

    curl \

    libpng-dev \

    libonig-dev \

    libxml2-dev \

    libzip-dev \

    zip \

    unzip \

    vim \

    wget

  

# Очистка кэша

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

  

# Установка PHP расширений

RUN docker-php-ext-install \

    pdo_mysql \

    mysqli \

    mbstring \

    exif \

    pcntl \

    bcmath \

    gd \

    zip \

    opcache

  

# Установка Composer

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

  

# Включение модулей Apache

RUN a2enmod rewrite headers expires

  

# Настройка прав доступа

RUN chown -R www-data:www-data /var/www/html \

    && chmod -R 755 /var/www/html

  

# Рабочая директория

WORKDIR /var/www/html

  

# Expose порт

EXPOSE 80

  

# Запуск Apache

CMD ["apache2-foreground"]

```

  

**apache/default.conf:**

  

```apache

<VirtualHost *:80>

    ServerName localhost

    DocumentRoot /var/www/html

  

    <Directory /var/www/html>

        Options Indexes FollowSymLinks

        AllowOverride All

        Require all granted

        # Включение .htaccess

        <IfModule mod_rewrite.c>

            RewriteEngine On

        </IfModule>

    </Directory>

  

    # Логи

    ErrorLog ${APACHE_LOG_DIR}/error.log

    CustomLog ${APACHE_LOG_DIR}/access.log combined

  

    # Настройки PHP

    php_value upload_max_filesize 50M

    php_value post_max_size 50M

    php_value memory_limit 256M

    php_value max_execution_time 300

</VirtualHost>

```

  

**php.ini:**

  

```ini

[PHP]

; Основные настройки

memory_limit = 256M

upload_max_filesize = 50M

post_max_size = 50M

max_execution_time = 300

max_input_time = 300

  

; Отображение ошибок (для разработки)

display_errors = On

display_startup_errors = On

error_reporting = E_ALL

  

; Логирование

log_errors = On

error_log = /var/log/php_errors.log

  

; Часовой пояс

date.timezone = Europe/Moscow

  

; OPcache (ускорение PHP)

opcache.enable = 1

opcache.memory_consumption = 128

opcache.interned_strings_buffer = 8

opcache.max_accelerated_files = 10000

opcache.revalidate_freq = 2

opcache.fast_shutdown = 1

```

  

**www/index.php:**

  

```php

<?php

phpinfo();

  

// Проверка подключения к MySQL

try {

    $pdo = new PDO(

        'mysql:host=db;dbname=myapp',

        'appuser',

        'apppassword'

    );

    echo "<h2>Подключение к MySQL: Успешно!</h2>";

} catch (PDOException $e) {

    echo "<h2>Ошибка подключения: " . $e->getMessage() . "</h2>";

}

?>

```

  

**### Вариант 2: Docker Compose с Nginx (LEMP)**

  

**docker-compose.yml (LEMP):**

  

```yaml

version: '3.8'

  

services:

  # Веб-сервер Nginx

  nginx:

    image: nginx:alpine

    container_name: lemp_nginx

    ports:

      - "8080:80"

    volumes:

      - ./www:/var/www/html

      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf

      - ./nginx/nginx.conf:/etc/nginx/nginx.conf

    depends_on:

      - php

      - db

    networks:

      - lemp_network

  

  # PHP-FPM

  php:

    build:

      context: .

      dockerfile: Dockerfile.php

    container_name: lemp_php

    volumes:

      - ./www:/var/www/html

      - ./php.ini:/usr/local/etc/php/php.ini

      - ./php-fpm/www.conf:/usr/local/etc/php-fpm.d/www.conf

    environment:

      - PHP_FPM_USER=www-data

      - PHP_FPM_GROUP=www-data

    networks:

      - lemp_network

  

  # База данных MySQL

  db:

    image: mysql:8.0

    container_name: lemp_mysql

    restart: always

    ports:

      - "3306:3306"

    environment:

      MYSQL_ROOT_PASSWORD: rootpassword

      MYSQL_DATABASE: myapp

      MYSQL_USER: appuser

      MYSQL_PASSWORD: apppassword

    volumes:

      - db_data:/var/lib/mysql

    networks:

      - lemp_network

  

  # Redis (кэш, сессии)

  redis:

    image: redis:alpine

    container_name: lemp_redis

    ports:

      - "6379:6379"

    networks:

      - lemp_network

  

volumes:

  db_data:

  

networks:

  lemp_network:

    driver: bridge

```

  

**Dockerfile.php:**

  

```dockerfile

FROM php:8.2-fpm

  

# Установка системных зависимостей

RUN apt-get update && apt-get install -y \

    git \

    curl \

    libpng-dev \

    libonig-dev \

    libxml2-dev \

    libzip-dev \

    libicu-dev \

    libfreetype6-dev \

    libjpeg62-turbo-dev \

    zip \

    unzip

  

# Установка PHP расширений

RUN docker-php-ext-configure gd --with-freetype --with-jpeg \

    && docker-php-ext-install \

    pdo_mysql \

    mysqli \

    mbstring \

    exif \

    pcntl \

    bcmath \

    gd \

    zip \

    intl \

    opcache

  

# Установка Redis расширения

RUN pecl install redis && docker-php-ext-enable redis

  

# Установка Composer

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

  

# Создание пользователя

RUN groupadd -g 1000 www-data-custom \

    && useradd -u 1000 -ms /bin/bash -g www-data-custom www-data-custom

  

# Права доступа

RUN chown -R www-data:www-data /var/www/html

  

WORKDIR /var/www/html

  

EXPOSE 9000

  

CMD ["php-fpm"]

```

  

**nginx/default.conf:**

  

```nginx

server {

    listen 80;

    server_name localhost;

    root /var/www/html;

    index index.php index.html index.htm;

  

    # Логи

    access_log /var/log/nginx/access.log;

    error_log /var/log/nginx/error.log;

  

    # Максимальный размер загружаемых файлов

    client_max_body_size 50M;

  

    # Основная location

    location / {

        try_files $uri $uri/ /index.php?$query_string;

    }

  

    # Обработка PHP

    location ~ \.php$ {

        try_files $uri =404;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;

        fastcgi_pass php:9000;

        fastcgi_index index.php;

        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;

        fastcgi_param PATH_INFO $fastcgi_path_info;

        # Таймауты

        fastcgi_read_timeout 300;

        fastcgi_send_timeout 300;

        fastcgi_connect_timeout 300;

    }

  

    # Отключение логирования для статики

    location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml|svg|woff|woff2|ttf|eot)$ {

        expires 30d;

        access_log off;

        log_not_found off;

    }

  

    # Запрет доступа к скрытым файлам

    location ~ /\. {

        deny all;

        access_log off;

        log_not_found off;

    }

}

```

  

**nginx/nginx.conf:**

  

```nginx

user www-data;

worker_processes auto;

pid /run/nginx.pid;

  

events {

    worker_connections 2048;

    multi_accept on;

    use epoll;

}

  

http {

    # Основные настройки

    sendfile on;

    tcp_nopush on;

    tcp_nodelay on;

    keepalive_timeout 65;

    types_hash_max_size 2048;

    server_tokens off;

  

    # Размеры буферов

    client_body_buffer_size 10K;

    client_header_buffer_size 1k;

    client_max_body_size 50M;

    large_client_header_buffers 4 16k;

  

    # Таймауты

    client_body_timeout 12;

    client_header_timeout 12;

    send_timeout 10;

  

    # MIME типы

    include /etc/nginx/mime.types;

    default_type application/octet-stream;

  

    # Логирование

    access_log /var/log/nginx/access.log;

    error_log /var/log/nginx/error.log;

  

    # Gzip сжатие

    gzip on;

    gzip_vary on;

    gzip_proxied any;

    gzip_comp_level 6;

    gzip_types text/plain text/css text/xml text/javascript 

               application/json application/javascript application/xml+rss 

               application/rss+xml font/truetype font/opentype 

               application/vnd.ms-fontobject image/svg+xml;

    gzip_disable "msie6";

  

    # Подключение конфигураций сайтов

    include /etc/nginx/conf.d/*.conf;

}

```

  

**php-fpm/www.conf:**

  

```ini

[www]

user = www-data

group = www-data

  

listen = 9000

  

; Настройки процессов

pm = dynamic

pm.max_children = 50

pm.start_servers = 10

pm.min_spare_servers = 5

pm.max_spare_servers = 20

pm.max_requests = 500

  

; Таймауты

request_terminate_timeout = 300s

  

; Логи

php_admin_value[error_log] = /var/log/php-fpm/www-error.log

php_admin_flag[log_errors] = on

  

; Переменные окружения

env[HOSTNAME] = $HOSTNAME

env[PATH] = /usr/local/bin:/usr/bin:/bin

env[TMP] = /tmp

env[TMPDIR] = /tmp

env[TEMP] = /tmp

```

  

**### Запуск Docker-окружения**

  

**Команды для запуска:**

  

```bash

# LAMP

cd /path/to/lamp-project

docker-compose up -d

  

# LEMP

cd /path/to/lemp-project

docker-compose up -d

  

# Проверка статуса

docker-compose ps

  

# Просмотр логов

docker-compose logs -f

  

# Вход в контейнер

docker-compose exec web bash          # для LAMP

docker-compose exec php bash          # для LEMP (PHP-FPM)

docker-compose exec nginx sh          # для LEMP (Nginx)

  

# Остановка

docker-compose stop

  

# Остановка и удаление

docker-compose down

  

# Остановка с удалением volumes

docker-compose down -v

```

  

**Полезные команды внутри контейнеров:**

  

```bash

# Проверка PHP

php -v

php -m  # список модулей

php -i  # phpinfo

  

# Проверка конфигурации Nginx

nginx -t

  

# Перезагрузка конфигурации Nginx

nginx -s reload

  

# Проверка конфигурации Apache

apache2ctl -t

apachectl configtest

  

# Просмотр логов PHP-FPM

tail -f /var/log/php-fpm/www-error.log

  

# Composer

composer install

composer update

composer require vendor/package

  

# Работа с MySQL

mysql -h db -u appuser -p

```

  

**### Вариант 3: Быстрый старт с готовыми образами**

  

**docker-compose.yml (минималистичный):**

  

```yaml

version: '3.8'

  

services:

  php:

    image: php:8.2-apache

    ports:

      - "8080:80"

    volumes:

      - ./www:/var/www/html

    environment:

      - APACHE_DOCUMENT_ROOT=/var/www/html

  

  db:

    image: mysql:8.0

    environment:

      MYSQL_ROOT_PASSWORD: root

      MYSQL_DATABASE: mydb

    volumes:

      - db_data:/var/lib/mysql

  

volumes:

  db_data:

```

  

**Запуск:**

  

```bash

# Создание структуры

mkdir lamp-quick && cd lamp-quick

mkdir www

  

# Создание index.php

echo "<?php phpinfo(); ?>" > www/index.php

  

# Создание docker-compose.yml

cat > docker-compose.yml << 'EOF'

# ... содержимое выше ...

EOF

  

# Запуск

docker-compose up -d

  

# Открытие в браузере

open http://localhost:8080

```

  

---

  

**## Установка на различные ОС**

  

**### Ubuntu/Debian (LAMP)**

  

**Установка Apache, MySQL, PHP:**

  

```bash

# Обновление системы

sudo apt update

sudo apt upgrade -y

  

# Установка Apache

sudo apt install apache2 -y

  

# Проверка статуса Apache

sudo systemctl status apache2

sudo systemctl enable apache2

  

# Установка MySQL

sudo apt install mysql-server -y

  

# Безопасная настройка MySQL

sudo mysql_secure_installation

  

# Установка PHP и модулей

sudo apt install php libapache2-mod-php php-mysql -y

  

# Установка дополнительных модулей PHP

sudo apt install php-cli php-fpm php-json php-common php-mysql \

    php-zip php-gd php-mbstring php-curl php-xml php-pear \

    php-bcmath php-intl php-imagick php-redis -y

  

# Перезапуск Apache

sudo systemctl restart apache2

  

# Проверка версий

apache2 -v

mysql --version

php -v

```

  

**Создание тестового сайта:**

  

```bash

# Создание директории

sudo mkdir -p /var/www/mysite

  

# Создание index.php

sudo nano /var/www/mysite/index.php

```

  

```php

<?php

phpinfo();

?>

```

  

**Настройка виртуального хоста:**

  

```bash

sudo nano /etc/apache2/sites-available/mysite.conf

```

  

```apache

<VirtualHost *:80>

    ServerName mysite.local

    ServerAdmin admin@mysite.local

    DocumentRoot /var/www/mysite

  

    <Directory /var/www/mysite>

        Options Indexes FollowSymLinks

        AllowOverride All

        Require all granted

    </Directory>

  

    ErrorLog ${APACHE_LOG_DIR}/mysite_error.log

    CustomLog ${APACHE_LOG_DIR}/mysite_access.log combined

</VirtualHost>

```

  

```bash

# Включение сайта

sudo a2ensite mysite.conf

  

# Включение mod_rewrite

sudo a2enmod rewrite

  

# Проверка конфигурации

sudo apache2ctl configtest

  

# Перезагрузка Apache

sudo systemctl reload apache2

  

# Добавление в /etc/hosts

echo "127.0.0.1 mysite.local" | sudo tee -a /etc/hosts

  

# Настройка прав

sudo chown -R www-data:www-data /var/www/mysite

sudo chmod -R 755 /var/www/mysite

```

  

**### Ubuntu/Debian (LEMP)**

  

```bash

# Обновление системы

sudo apt update && sudo apt upgrade -y

  

# Установка Nginx

sudo apt install nginx -y

sudo systemctl start nginx

sudo systemctl enable nginx

  

# Установка MySQL

sudo apt install mysql-server -y

sudo mysql_secure_installation

  

# Установка PHP-FPM

sudo apt install php-fpm php-mysql -y

  

# Установка дополнительных модулей

sudo apt install php-cli php-json php-common php-curl \

    php-zip php-gd php-mbstring php-xml php-bcmath \

    php-intl php-redis -y

  

# Проверка версии PHP-FPM

php-fpm8.2 -v  # версия может отличаться

```

  

**Настройка сайта для Nginx:**

  

```bash

sudo nano /etc/nginx/sites-available/mysite

```

  

```nginx

server {

    listen 80;

    server_name mysite.local;

    root /var/www/mysite;

    index index.php index.html;

  

    location / {

        try_files $uri $uri/ =404;

    }

  

    location ~ \.php$ {

        include snippets/fastcgi-php.conf;

        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;

    }

  

    location ~ /\.ht {

        deny all;

    }

}

```

  

```bash

# Создание symlink

sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

  

# Проверка конфигурации

sudo nginx -t

  

# Перезагрузка Nginx

sudo systemctl reload nginx

  

# Создание директории и тестового файла

sudo mkdir -p /var/www/mysite

echo "<?php phpinfo(); ?>" | sudo tee /var/www/mysite/index.php

  

# Права доступа

sudo chown -R www-data:www-data /var/www/mysite

sudo chmod -R 755 /var/www/mysite

  

# Добавление в hosts

echo "127.0.0.1 mysite.local" | sudo tee -a /etc/hosts

```

  

**### CentOS/RHEL/Rocky Linux (LAMP)**

  

```bash

# Обновление системы

sudo dnf update -y

  

# Установка Apache

sudo dnf install httpd -y

sudo systemctl start httpd

sudo systemctl enable httpd

  

# Открытие портов в firewall

sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --reload

  

# Установка MySQL (MariaDB)

sudo dnf install mariadb-server -y

sudo systemctl start mariadb

sudo systemctl enable mariadb

sudo mysql_secure_installation

  

# Установка PHP

sudo dnf install php php-mysqlnd php-fpm php-opcache php-gd \

    php-xml php-mbstring php-json php-cli php-curl \

    php-zip php-bcmath php-intl -y

  

# Перезапуск Apache

sudo systemctl restart httpd

  

# Проверка версий

httpd -v

mysql --version

php -v

```

  

**### macOS (LAMP)**

  

**Использование Homebrew:**

  

```bash

# Установка Homebrew (если не установлен)

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

  

# Установка Apache

brew install httpd

  

# Запуск Apache

brew services start httpd

  

# Установка MySQL

brew install mysql

brew services start mysql

  

# Безопасная настройка

mysql_secure_installation

  

# Установка PHP

brew install php

brew install php@8.2  # конкретная версия

  

# Связывание PHP с Apache

brew install php@8.2

echo "export PATH=\"/opt/homebrew/opt/php@8.2/bin:\$PATH\"" >> ~/.zshrc

source ~/.zshrc

  

# Установка дополнительных расширений

pecl install redis

pecl install xdebug

  

# Редактирование конфигурации Apache

nano /opt/homebrew/etc/httpd/httpd.conf

```

  

**Конфигурация httpd.conf для PHP:**

  

```apache

LoadModule php_module /opt/homebrew/opt/php@8.2/lib/httpd/modules/libphp.so

  

<FilesMatch \.php$>

    SetHandler application/x-httpd-php

</FilesMatch>

  

<IfModule dir_module>

    DirectoryIndex index.php index.html

</IfModule>

```

  

```bash

# Перезапуск Apache

brew services restart httpd

  

# Проверка

open http://localhost:8080

```

  

**### Windows (LAMP через XAMPP)**

  

**XAMPP — готовая сборка Apache, MySQL, PHP, Perl:**

  

```

1. Скачать XAMPP с официального сайта:

   https://www.apachefriends.org/

  

2. Запустить установщик

   - Выбрать компоненты: Apache, MySQL, PHP, phpMyAdmin

   - Указать путь установки (например, C:\xampp)

  

3. Запустить XAMPP Control Panel

   - Запустить Apache (кнопка Start)

   - Запустить MySQL (кнопка Start)

  

4. Проверить работу:

   - Открыть http://localhost

   - Должна открыться стартовая страница XAMPP

  

5. Создать проект:

   C:\xampp\htdocs\myproject\index.php

```

  

**Настройка виртуальных хостов в XAMPP:**

  

```apache

# C:\xampp\apache\conf\extra\httpd-vhosts.conf

  

<VirtualHost *:80>

    ServerName myproject.local

    DocumentRoot "C:/xampp/htdocs/myproject"

    <Directory "C:/xampp/htdocs/myproject">

        Options Indexes FollowSymLinks

        AllowOverride All

        Require all granted

    </Directory>

</VirtualHost>

```

  

```

# C:\Windows\System32\drivers\etc\hosts

127.0.0.1 myproject.local

```

  

**Альтернатива: WampServer, Laragon**

  

```

WampServer: https://www.wampserver.com/

Laragon: https://laragon.org/ (рекомендуется, очень удобный)

```

  

---

  

**## Способы запуска PHP**

  

**### 1. mod_php (Apache Module)**

  

**Описание:**

PHP загружается как модуль Apache, встраивается непосредственно в процесс веб-сервера.

  

```apache

LoadModule php_module modules/libphp.so

  

<FilesMatch \.php$>

    SetHandler application/x-httpd-php

</FilesMatch>

```

  

**Преимущества:**

- Простота настройки

- Низкая задержка (PHP в том же процессе)

- Не требует дополнительных служб

  

**Недостатки:**

- Высокое потребление памяти (каждый процесс Apache содержит PHP)

- Невозможность использовать разные версии PHP

- Проблемы с правами доступа

- Устаревший подход

  

**Когда использовать:**

- Простые проекты

- Shared-хостинг

- Legacy-приложения

  

**### 2. PHP-FPM (FastCGI Process Manager)**

  

**Описание:**

PHP работает как отдельный процесс-менеджер, веб-сервер общается с ним через FastCGI протокол.

  

```

Nginx/Apache → FastCGI → PHP-FPM → PHP Worker Pool

```

  

**Установка PHP-FPM:**

  

```bash

# Ubuntu/Debian

sudo apt install php-fpm

  

# CentOS/RHEL

sudo dnf install php-fpm

  

# Запуск

sudo systemctl start php8.2-fpm

sudo systemctl enable php8.2-fpm

```

  

**Конфигурация pool'а (/etc/php/8.2/fpm/pool.d/www.conf):**

  

```ini

[www]

user = www-data

group = www-data

  

; Способ подключения: unix socket (быстрее) или TCP

listen = /run/php/php8.2-fpm.sock

; listen = 127.0.0.1:9000

  

listen.owner = www-data

listen.group = www-data

listen.mode = 0660

  

; Управление процессами

pm = dynamic

pm.max_children = 50        ; Максимум дочерних процессов

pm.start_servers = 10       ; Сколько запустить при старте

pm.min_spare_servers = 5    ; Минимум свободных процессов

pm.max_spare_servers = 20   ; Максимум свободных процессов

pm.max_requests = 500       ; Перезапуск после N запросов

  

; Таймауты

request_terminate_timeout = 300s

request_slowlog_timeout = 10s

  

; Логирование медленных запросов

slowlog = /var/log/php-fpm/www-slow.log

  

; Мониторинг

pm.status_path = /status

ping.path = /ping

```

  

**Типы управления процессами (pm):**

  

```ini

; static - фиксированное количество процессов

pm = static

pm.max_children = 20

  

; dynamic - динамическое создание/удаление

pm = dynamic

pm.max_children = 50

pm.start_servers = 10

pm.min_spare_servers = 5

pm.max_spare_servers = 20

  

; ondemand - создаются по требованию

pm = ondemand

pm.max_children = 50

pm.process_idle_timeout = 10s

```

  

**Расчет pm.max_children:**

  

```bash

# Формула: (Доступная RAM) / (Средний размер PHP процесса)

  

# Узнать средний размер процесса

ps aux | grep php-fpm | awk '{sum+=$6} END {print sum/NR/1024 " MB"}'

  

# Пример:

# Доступно RAM: 4GB (оставим 1GB для системы = 3GB = 3072MB)

# Средний размер процесса: 30MB

# pm.max_children = 3072 / 30 ≈ 100

```

  

**Мониторинг PHP-FPM:**

  

```bash

# Статус через веб

curl http://localhost/status

curl http://localhost/ping

  

# Логи

tail -f /var/log/php8.2-fpm.log

tail -f /var/log/php-fpm/www-slow.log

  

# Активные процессы

ps aux | grep php-fpm

  

# Детальная статистика

sudo systemctl status php8.2-fpm

```

  

**Преимущества:**

- Лучшая изоляция процессов

- Экономия памяти

- Возможность использовать разные версии PHP

- Гибкая настройка пулов процессов

- Перезапуск без остановки веб-сервера

  

**Недостатки:**

- Чуть более сложная настройка

- Дополнительный процесс

  

**Когда использовать:**

- Современные проекты

- Nginx (обязательно)

- Высоконагруженные приложения

- Микросервисы

  

**### 3. PHP-CGI (Common Gateway Interface)**

  

**Описание:**

Устаревший способ — для каждого запроса запускается отдельный процесс PHP.

  

```bash

# Запуск через Apache

ScriptAlias /php-cgi /usr/local/bin/php-cgi

```

  

**Недостатки:**

- Очень медленный (новый процесс на каждый запрос)

- Высокая нагрузка на систему

- Устаревший

  

**Не рекомендуется использовать.**

  

**### 4. PHP-CLI (Command Line Interface)**

  

**Описание:**

Запуск PHP скриптов из командной строки.

  

```bash

# Выполнение скрипта

php script.php

  

# С аргументами

php script.php arg1 arg2

  

# Интерактивный режим

php -a

  

# Встроенный веб-сервер (только для разработки!)

php -S localhost:8000

  

# С указанием директории

php -S localhost:8000 -t /path/to/public

  

# С роутингом

php -S localhost:8000 router.php

```

  

**Примеры использования:**

  

```php

<?php

// script.php

echo "Hello from CLI!\n";

echo "Arguments: " . implode(', ', $argv) . "\n";

?>

```

  

```bash

php script.php test 123

# Вывод:

# Hello from CLI!

# Arguments: script.php, test, 123

```

  

**Для долгоиграющих процессов (воркеры очередей):**

  

```php

<?php

// worker.php

while (true) {

    // Обработка задачи из очереди

    processJob();

    // Пауза

    sleep(1);

    // Освобождение памяти

    gc_collect_cycles();

}

?>

```

  

```bash

# Запуск в фоне

nohup php worker.php > worker.log 2>&1 &

  

# Через systemd

sudo nano /etc/systemd/system/php-worker.service

```

  

```ini

[Unit]

Description=PHP Worker

After=network.target

  

[Service]

Type=simple

User=www-data

WorkingDirectory=/var/www/app

ExecStart=/usr/bin/php /var/www/app/worker.php

Restart=always

  

[Install]

WantedBy=multi-user.target

```

  

```bash

sudo systemctl daemon-reload

sudo systemctl start php-worker

sudo systemctl enable php-worker

```

  

**Преимущества:**

- Подходит для cron-задач

- Воркеры очередей

- Скрипты обслуживания

- Тестирование

  

**### 5. Встроенный веб-сервер PHP**

  

**Только для разработки!**

  

```bash

# Простой запуск

php -S localhost:8000

  

# С указанием публичной директории

cd /path/to/project

php -S localhost:8000 -t public

  

# С кастомным роутером

php -S localhost:8000 router.php

```

  

**router.php:**

  

```php

<?php

// router.php

$uri = $_SERVER['REQUEST_URI'];

  

// Статические файлы отдаем как есть

if (preg_match('/\.(?:png|jpg|jpeg|gif|css|js)$/', $uri)) {

    return false;

}

  

// Все остальное на index.php

require_once __DIR__ . '/public/index.php';

?>

```

  

**Преимущества:**

- Не требует установки веб-сервера

- Быстрый старт для разработки

- Отладка простых скриптов

  

**Недостатки:**

- Однопоточный (медленный)

- Нет продвинутых функций

- Не для production

  

**### Сравнение способов запуска**

  

| Способ | Производительность | Память | Сложность | Production |

|--------|-------------------|--------|-----------|------------|

| mod_php | Средняя | Высокая | Низкая | ⚠️ Устарел |

| PHP-FPM | Высокая | Низкая | Средняя | ✅ Да |

| PHP-CGI | Низкая | Высокая | Низкая | ❌ Нет |

| PHP-CLI | - | Низкая | Низкая | ✅ Для задач |

| Встроенный | Низкая | Низкая | Очень низкая | ❌ Только dev |

  

---

  

**## Пользователи и права доступа**

  

**### От имени какого пользователя должен работать веб-сервер?**

  

**Основные принципы безопасности:**

  

1. **Веб-сервер не должен работать от root**

2. **Использовать специального пользователя с минимальными правами**

3. **Разделять пользователей веб-сервера и PHP**

4. **Правильно настраивать права доступа к файлам**

  

**### Стандартные пользователи**

  

**Linux (Ubuntu/Debian):**

```bash

# Apache

User: www-data

Group: www-data

  

# Nginx

User: www-data

Group: www-data

  

# PHP-FPM

User: www-data

Group: www-data

```

  

**Linux (CentOS/RHEL):**

```bash

# Apache

User: apache

Group: apache

  

# Nginx

User: nginx

Group: nginx

  

# PHP-FPM

User: apache (или nginx)

Group: apache (или nginx)

```

  

**macOS:**

```bash

# Apache

User: _www

Group: _www

```

  

**### Проверка текущих пользователей**

  

```bash

# Apache

ps aux | grep apache

cat /etc/apache2/apache2.conf | grep -E "^User|^Group"

  

# Nginx

ps aux | grep nginx

cat /etc/nginx/nginx.conf | grep -E "^user"

  

# PHP-FPM

ps aux | grep php-fpm

cat /etc/php/8.2/fpm/pool.d/www.conf | grep -E "^user|^group"

```

  

**### Настройка прав доступа к файлам**

  

**Рекомендуемая схема прав:**

  

```

Директории: 755 (rwxr-xr-x)

Файлы: 644 (rw-r--r--)

Владелец: developer (ваш пользователь)

Группа: www-data (группа веб-сервера)

```

  

**Настройка прав:**

  

```bash

# Изменение владельца

sudo chown -R developer:www-data /var/www/mysite

  

# Права на директории

sudo find /var/www/mysite -type d -exec chmod 755 {} \;

  

# Права на файлы

sudo find /var/www/mysite -type f -exec chmod 644 {} \;

  

# Для upload/cache директорий (нужна запись)

sudo chmod -R 775 /var/www/mysite/storage

sudo chmod -R 775 /var/www/mysite/var/cache

  

# Проверка прав

ls -la /var/www/mysite

```

  

**Объяснение прав:**

  

```

755 = rwxr-xr-x

7 (rwx) - владелец: читать, писать, выполнять

5 (r-x) - группа: читать, выполнять

5 (r-x) - остальные: читать, выполнять

  

644 = rw-r--r--

6 (rw-) - владелец: читать, писать

4 (r--) - группа: читать

4 (r--) - остальные: читать

  

775 = rwxrwxr-x

7 (rwx) - владелец: читать, писать, выполнять

7 (rwx) - группа: читать, писать, выполнять

5 (r-x) - остальные: читать, выполнять

```

  

**### Продвинутая настройка: разные пользователи**

  

**Сценарий:** Разработчик и веб-сервер используют разные пользователи.

  

```bash

# Создание группы для проекта

sudo groupadd webprojects

  

# Добавление пользователей в группу

sudo usermod -aG webprojects developer

sudo usermod -aG webprojects www-data

  

# Установка владельца и группы

sudo chown -R developer:webprojects /var/www/mysite

  

# Права с setgid (новые файлы наследуют группу)

sudo chmod g+s /var/www/mysite

  

# Права на директории и файлы

sudo find /var/www/mysite -type d -exec chmod 2775 {} \;

sudo find /var/www/mysite -type f -exec chmod 664 {} \;

```

  

**Проверка:**

  

```bash

# Проверка членства в группах

groups developer

groups www-data

  

# Проверка прав

ls -la /var/www/mysite

  

# Тест создания файла

sudo -u developer touch /var/www/mysite/test.txt

ls -l /var/www/mysite/test.txt

# Должно быть: -rw-rw-r-- developer webprojects

  

# Тест записи от веб-сервера

sudo -u www-data touch /var/www/mysite/test2.txt

ls -l /var/www/mysite/test2.txt

```

  

**### ACL (Access Control Lists) — продвинутые права**

  

**Установка ACL:**

  

```bash

# Ubuntu/Debian

sudo apt install acl

  

# Проверка поддержки ACL

mount | grep acl

```

  

**Использование ACL:**

  

```bash

# Установка ACL для пользователя

sudo setfacl -R -m u:www-data:rwx /var/www/mysite/storage

  

# Установка ACL для группы

sudo setfacl -R -m g:www-data:rwx /var/www/mysite/var

  

# Установка default ACL (для новых файлов)

sudo setfacl -R -d -m u:www-data:rwx /var/www/mysite/storage

sudo setfacl -R -d -m g:www-data:rwx /var/www/mysite/var

  

# Просмотр ACL

getfacl /var/www/mysite/storage

  

# Удаление ACL

sudo setfacl -R -b /var/www/mysite/storage

```

  

**Пример вывода getfacl:**

  

```

# file: storage/

# owner: developer

# group: www-data

user::rwx

user:www-data:rwx

group::r-x

mask::rwx

other::r-x

default:user::rwx

default:user:www-data:rwx

default:group::r-x

default:mask::rwx

default:other::r-x

```

  

**### SELinux (CentOS/RHEL)**

  

```bash

# Проверка статуса SELinux

getenforce

  

# Установка контекста для веб-файлов

sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/mysite(/.*)?"

sudo restorecon -Rv /var/www/mysite

  

# Для директорий с записью

sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/mysite/storage(/.*)?"

sudo restorecon -Rv /var/www/mysite/storage

  

# Разрешение сетевых подключений для PHP

sudo setsebool -P httpd_can_network_connect 1

sudo setsebool -P httpd_can_network_connect_db 1

  

# Просмотр контекста

ls -Z /var/www/mysite

```

  

**### Безопасность: что НЕ надо делать**

  

❌ **Плохие практики:**

  

```bash

# НЕ ДЕЛАЙТЕ ТАК!

  

# 777 на все файлы (любой может писать)

chmod -R 777 /var/www/mysite  # ОПАСНО!

  

# Запуск веб-сервера от root

User root  # ОПАСНО!

  

# Владелец www-data (веб-сервер может изменять код)

chown -R www-data:www-data /var/www/mysite  # ПЛОХО!

  

# Отключение SELinux

setenforce 0  # НЕБЕЗОПАСНО!

```

  

**### Типичная структура прав для фреймворков**

  

**Laravel:**

  

```bash

# Владелец и группа

sudo chown -R developer:www-data /var/www/laravel

  

# Базовые права

sudo chmod -R 755 /var/www/laravel

  

# Директории с записью

sudo chmod -R 775 /var/www/laravel/storage

sudo chmod -R 775 /var/www/laravel/bootstrap/cache

  

# Через ACL (рекомендуется)

sudo setfacl -R -m u:www-data:rwx /var/www/laravel/storage

sudo setfacl -R -m u:www-data:rwx /var/www/laravel/bootstrap/cache

sudo setfacl -R -d -m u:www-data:rwx /var/www/laravel/storage

sudo setfacl -R -d -m u:www-data:rwx /var/www/laravel/bootstrap/cache

```

  

**Symfony:**

  

```bash

sudo chown -R developer:www-data /var/www/symfony

sudo chmod -R 755 /var/www/symfony

sudo chmod -R 775 /var/www/symfony/var

sudo setfacl -R -m u:www-data:rwx /var/www/symfony/var

sudo setfacl -R -d -m u:www-data:rwx /var/www/symfony/var

```

  

---

  

**## Оптимизация и безопасность**

  

**### Оптимизация PHP**

  

**php.ini настройки:**

  

```ini

; Память

memory_limit = 256M              ; Увеличить для сложных приложений

  

; Загрузка файлов

upload_max_filesize = 50M

post_max_size = 50M

max_file_uploads = 20

  

; Время выполнения

max_execution_time = 300

max_input_time = 300

  

; OPcache (обязательно в production!)

opcache.enable = 1

opcache.memory_consumption = 256

opcache.interned_strings_buffer = 16

opcache.max_accelerated_files = 20000

opcache.revalidate_freq = 60        ; Проверка изменений каждые 60 сек

opcache.fast_shutdown = 1

opcache.enable_cli = 0

  

; Realpath cache (ускорение работы с файлами)

realpath_cache_size = 4M

realpath_cache_ttl = 600

  

; JIT (PHP 8+)

opcache.jit = tracing

opcache.jit_buffer_size = 128M

```

  

**### Безопасность PHP**

  

```ini

; Отключение опасных функций

disable_functions = exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source

  

; Отключение ошибок в production

display_errors = Off

display_startup_errors = Off

log_errors = On

error_log = /var/log/php_errors.log

  

; Безопасность

expose_php = Off                    ; Скрыть версию PHP

allow_url_fopen = Off              ; Запретить удаленные файлы

allow_url_include = Off

enable_dl = Off

file_uploads = On

max_input_vars = 3000

session.cookie_httponly = 1

session.cookie_secure = 1          ; Только HTTPS

session.use_strict_mode = 1

```

  

**### Оптимизация MySQL**

  

**/etc/mysql/my.cnf:**

  

```ini

[mysqld]

# InnoDB настройки

innodb_buffer_pool_size = 1G       ; 70-80% доступной RAM

innodb_log_file_size = 256M

innodb_flush_log_at_trx_commit = 2

innodb_flush_method = O_DIRECT

  

# Query cache (MySQL 5.7, убран в 8.0)

query_cache_type = 1

query_cache_size = 64M

query_cache_limit = 2M

  

# Connections

max_connections = 200

max_connect_errors = 100

  

# Slow query log

slow_query_log = 1

slow_query_log_file = /var/log/mysql/slow.log

long_query_time = 2

```

  

**### Мониторинг**

  

**Установка инструментов мониторинга:**

  

```bash

# Netdata (комплексный мониторинг)

bash <(curl -Ss https://my-netdata.io/kickstart.sh)

  

# Открыть в браузере

open http://localhost:19999

  

# Glances (консольный мониторинг)

sudo apt install glances

glances

  

# Для PHP-FPM status page

curl http://localhost/status?full

```

  

**Логи для анализа:**

  

```bash

# Apache

tail -f /var/log/apache2/access.log

tail -f /var/log/apache2/error.log

  

# Nginx

tail -f /var/log/nginx/access.log

tail -f /var/log/nginx/error.log

  

# PHP-FPM

tail -f /var/log/php8.2-fpm.log

tail -f /var/log/php-fpm/www-slow.log

  

# MySQL

tail -f /var/log/mysql/error.log

tail -f /var/log/mysql/slow.log

  

# Анализ логов с помощью GoAccess

sudo apt install goaccess

goaccess /var/log/nginx/access.log -o report.html --log-format=COMBINED

```

  

---

  

**## Заключение**

  

**### Чек-лист для production**

  

✅ **Веб-сервер:**

- [ ] Настроен виртуальный хост

- [ ] Включен HTTPS (Let's Encrypt)

- [ ] Настроено кэширование статики

- [ ] Включено Gzip/Brotli сжатие

- [ ] Настроены rate limits

- [ ] Скрыта версия сервера

  

✅ **PHP:**

- [ ] Включен OPcache

- [ ] Отключены ошибки в браузере

- [ ] Настроено логирование

- [ ] Отключены опасные функции

- [ ] Правильно настроен PHP-FPM pool

  

✅ **База данных:**

- [ ] Изменен пароль root

- [ ] Созданы отдельные пользователи для приложений

- [ ] Настроен автоматический бэкап

- [ ] Оптимизированы настройки памяти

- [ ] Включен slow query log

  

✅ **Безопасность:**

- [ ] Файрволл настроен (ufw/firewalld)

- [ ] Fail2ban установлен

- [ ] SSH доступ по ключу

- [ ] Регулярные обновления безопасности

- [ ] Правильные права на файлы

  

✅ **Мониторинг:**

- [ ] Установлен мониторинг (Netdata/Prometheus)

- [ ] Настроены алерты

- [ ] Логи ротируются

- [ ] Метрики собираются

  

**### Полезные ресурсы**