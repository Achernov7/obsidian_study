---

tags:

- php

- debugging

- xdebug

- tools

date: 2026-05-26

---

  

# xDebug

  

## Что такое xDebug

  

xDebug — это расширение PHP, предназначенное для отладки и профилирования PHP-приложений. Это один из самых популярных и мощных инструментов для разработчиков PHP, который значительно упрощает процесс поиска и исправления ошибок в коде.

  

## Основные возможности xDebug

  

### 1. Отладка кода (Debugging)

  

- **Пошаговое выполнение** — возможность выполнять код строка за строкой

- **Точки останова (Breakpoints)** — остановка выполнения в определенных местах

- **Просмотр переменных** — инспекция значений переменных во время выполнения

- **Стек вызовов** — отслеживание последовательности вызовов функций

- **Удаленная отладка** — возможность отлаживать код на удаленном сервере

  

### 2. Профилирование производительности

  

- **Анализ времени выполнения** функций и методов

- **Подсчет вызовов** функций

- **Анализ использования памяти**

- **Генерация отчетов** в формате Cachegrind

  

### 3. Трассировка выполнения

  

- **Детальное логирование** выполнения скрипта

- **Отслеживание вызовов функций** с параметрами и возвращаемыми значениями

- **Анализ потока выполнения**

  

### 4. Улучшенная обработка ошибок

  

- **Детальные стеки ошибок** с подсветкой синтаксиса

- **Форматированный вывод** `var_dump()`

- **Цветное выделение** в консоли

  

---

  

## Установка xDebug

  

### Установка через PECL

  

```bash

# Установка последней версии

pecl install xdebug

  

# Установка конкретной версии

pecl install xdebug-3.2.0

```

  

### Установка через менеджеры пакетов

  

#### Ubuntu/Debian

  

```bash

# PHP 8.x

sudo apt-get install php8.1-xdebug

  

# PHP 7.x

sudo apt-get install php7.4-xdebug

```

  

#### macOS (Homebrew)

  

```bash

# Если PHP установлен через Homebrew

brew install php@8.1-xdebug

```

  

#### Windows

  

1. Скачать DLL с официального сайта [xdebug.org](https://xdebug.org)

2. Поместить в папку extensions PHP

3. Добавить в `php.ini`

  

### Компиляция из исходников

  

```bash

git clone https://github.com/xdebug/xdebug.git

cd xdebug

phpize

./configure --enable-xdebug

make

sudo make install

```

  

---

  

## Конфигурация xDebug

  

### Базовая конфигурация в php.ini

  

```ini

; Включение расширения

zend_extension=xdebug

  

; Основные настройки xDebug 3.x

xdebug.mode=debug,develop,trace,profile

xdebug.client_host=localhost

xdebug.client_port=9003

xdebug.start_with_request=yes

  

; Настройки для IDE

xdebug.idekey=PHPSTORM

  

; Логирование

xdebug.log=/tmp/xdebug.log

xdebug.log_level=7

  

; Настройки вывода ошибок

xdebug.show_error_trace=1

xdebug.show_exception_trace=1

xdebug.show_local_vars=1

  

; Улучшенный var_dump

xdebug.var_display_max_children=256

xdebug.var_display_max_data=1024

xdebug.var_display_max_depth=5

  

; Профилирование

xdebug.output_dir=/tmp/xdebug

xdebug.profiler_output_name=cachegrind.out.%p

  

; Трассировка

xdebug.trace_output_dir=/tmp/xdebug

xdebug.trace_output_name=trace.%c

xdebug.trace_format=0

```

  

### Режимы работы xDebug 3.x

  

#### `debug`

  

Основной режим для пошаговой отладки.

  

```ini

xdebug.mode=debug

xdebug.start_with_request=trigger

```

  

#### `develop`

  

Режим разработки с улучшенными сообщениями об ошибках.

  

```ini

xdebug.mode=develop

```

  

#### `coverage`

  

Для анализа покрытия кода тестами.

  

```ini

xdebug.mode=coverage

```

  

#### `profile`

  

Для профилирования производительности.

  

```ini

xdebug.mode=profile

xdebug.start_with_request=trigger

```

  

#### `trace`

  

Для трассировки выполнения.

  

```ini

xdebug.mode=trace

xdebug.start_with_request=trigger

```

  

---

  

## Настройка IDE для работы с xDebug

  

### PhpStorm

  

1. **File → Settings → PHP → Debug**

- Установить порт: `9003` (xDebug 3.x) или `9000` (xDebug 2.x)

- Включить *Can accept external connections*

2. **Настройка сервера**

- File → Settings → PHP → Servers

- Добавить новый сервер с правильным хостом и портом

- Настроить **path mapping** для удаленной отладки

3. **Запуск отладки**

- Установить breakpoint (щелчок на номере строки)

- Нажать *Start Listening for PHP Debug Connections*

- Загрузить страницу в браузере с параметром `XDEBUG_SESSION_START`

  

### VS Code

  

1. Установить расширение **PHP Debug** от Derick Rethans

2. Конфигурация `launch.json`:

  

```json

{

"version": "0.2.0",

"configurations": [

{

"name": "Listen for Xdebug",

"type": "php",

"request": "launch",

"port": 9003,

"pathMappings": {

"/var/www/html": "${workspaceFolder}"

}

}

]

}

```

  

### Sublime Text

  

1. Установить пакет **Xdebug Client**

2. Настройка:

  

```json

{

"port": 9003,

"max_children": 32,

"max_data": 1024,

"max_depth": 4,

"pretty_output": true,

"path_mapping": {

"/var/www/html": "/local/path/to/project"

}

}

```

  

---

  

## Практическое использование

  

### Базовая отладка

  

#### Установка breakpoint в коде

  

```php

function calculateSum($a, $b) {

$result = $a + $b; // ← breakpoint здесь

return $result;

}

  

$sum = calculateSum(5, 10);

echo $sum;

```

  

#### Условные breakpoints

  

```php

for ($i = 0; $i < 100; $i++) {

// Breakpoint сработает только когда $i === 50

if ($i === 50) {

echo "Debug point reached"; // ← breakpoint здесь

}

}

```

  

### Отладка веб-приложений

  

#### Активация через URL-параметры

  

```

# Активация сессии отладки

http://localhost/script.php?XDEBUG_SESSION_START=PHPSTORM

  

# Остановка сессии отладки

http://localhost/script.php?XDEBUG_SESSION_STOP=PHPSTORM

```

  

#### Активация через cookies

  

```javascript

document.cookie = "XDEBUG_SESSION=PHPSTORM; path=/";

```

  

#### Отладка AJAX-запросов

  

```php

// api.php

header('Content-Type: application/json');

  

$data = json_decode(file_get_contents('php://input'), true);

$response = processApiRequest($data);

  

echo json_encode($response);

```

  

### Профилирование производительности

  

#### Активация профилирования

  

```ini

xdebug.mode=profile

xdebug.start_with_request=trigger

xdebug.trigger_value=profile_me

```

  

```

# URL для активации профилирования

http://localhost/script.php?XDEBUG_PROFILE=profile_me

```

  

#### Анализ результатов профилирования

  

```bash

# Установка KCacheGrind (Linux)

sudo apt-get install kcachegrind

  

# Установка QCacheGrind (macOS)

brew install qcachegrind

  

# Открытие файла профилирования

kcachegrind /tmp/xdebug/cachegrind.out.12345

```

  

### Трассировка выполнения

  

#### Конфигурация трассировки

  

```ini

xdebug.mode=trace

xdebug.start_with_request=trigger

xdebug.trace_format=0

xdebug.trace_options=1

xdebug.collect_params=4

xdebug.collect_return=1

```

  

#### Пример файла трассировки

  

```

TRACE START [2023-11-05 10:30:00]

0.0001 114928 -> {main}() /var/www/html/script.php:0

0.0002 115056 -> calculateSum() /var/www/html/script.php:8

0.0003 115088 -> floatval() /var/www/html/script.php:3

0.0004 115120 <- calculateSum() /var/www/html/script.php:8

0.0005 114984 <- {main}() /var/www/html/script.php:0

TRACE END [2023-11-05 10:30:00]

```

  

---

  

## Отладка различных типов приложений

  

### CLI-скрипты

  

```ini

xdebug.mode=debug

xdebug.start_with_request=yes

```

  

```bash

php -dxdebug.start_with_request=yes script.php

```

  

### Symfony

  

```yaml

# config/packages/dev/framework.yaml

framework:

profiler:

enabled: true

```

  

```php

// src/Controller/DebugController.php

namespace App\Controller;

  

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

  

class DebugController extends AbstractController

{

public function index()

{

$data = $this->getData(); // ← breakpoint здесь

return $this->json($data);

}

}

```

  

### Laravel

  

```php

// routes/web.php

Route::get('/debug', function () {

$users = App\Models\User::all(); // ← breakpoint здесь

return view('debug', compact('users'));

});

```

  

```php

// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

  

class UserController extends Controller

{

public function index()

{

$query = User::query();

  

if (request()->has('filter')) {

$query->where('name', 'like', '%' . request('filter') . '%');

}

  

$users = $query->get(); // ← breakpoint здесь

return view('users.index', compact('users'));

}

}

```

  

### WordPress

  

```php

// wp-config.php

define('WP_DEBUG', true);

define('WP_DEBUG_LOG', true);

  

// В теме или плагине

function debug_hook() {

$post = get_post(); // ← breakpoint здесь

}

add_action('wp_head', 'debug_hook');

```

  

---

  

## Распространенные проблемы и решения

  

### xDebug не подключается

  

```bash

# Проверка загрузки расширения

php -m | grep xdebug

  

# Проверка конфигурации

php -i | grep xdebug

  

# Проверка доступности порта

telnet localhost 9003

  

# Проверка с помощью netstat

netstat -an | grep 9003

```

  

### Медленная работа с xDebug

  

```ini

xdebug.mode=debug

xdebug.start_with_request=trigger

  

xdebug.var_display_max_depth=3

xdebug.var_display_max_children=128

xdebug.var_display_max_data=512

```

  

### Конфликт с production

  

Условная загрузка через environment:

  

```ini

; Только для development

[development]

zend_extension=xdebug

xdebug.mode=debug,develop

```

  

```bash

# .env

XDEBUG_MODE=debug

XDEBUG_CLIENT_HOST=localhost

```

  

```php

if ($_ENV['APP_ENV'] === 'development') {

ini_set('xdebug.mode', 'debug');

}

```

  

---

  

## Интеграция с тестированием

  

### PHPUnit с покрытием кода

  

```ini

xdebug.mode=coverage

```

  

```xml

<!-- phpunit.xml -->

<phpunit>

<coverage>

<include>

<directory suffix=".php">src</directory>

</include>

<report>

<html outputDirectory="coverage-html"/>

<text outputFile="coverage.txt"/>

</report>

</coverage>

</phpunit>

```

  

```bash

./vendor/bin/phpunit --coverage-html coverage/

```

  

### Отладка unit-тестов

  

```php

use PHPUnit\Framework\TestCase;

  

class CalculatorTest extends TestCase

{

public function testAddition()

{

$calculator = new Calculator();

$result = $calculator->add(2, 3); // ← breakpoint здесь

$this->assertEquals(5, $result);

}

}

```

  

---

  

## Продвинутые техники

  

### Удаленная отладка через SSH-туннель

  

```bash

ssh -R 9003:localhost:9003 user@remote-server.com

```

  

На удаленном сервере:

  

```ini

xdebug.client_host=127.0.0.1

xdebug.client_port=9003

```

  

### Отладка в Docker-контейнерах

  

```dockerfile

FROM php:8.1-apache

  

RUN pecl install xdebug \

&& docker-php-ext-enable xdebug

  

COPY xdebug.ini /usr/local/etc/php/conf.d/

```

  

```ini

# xdebug.ini для Docker

xdebug.mode=debug

xdebug.client_host=host.docker.internal

xdebug.client_port=9003

xdebug.start_with_request=yes

```

  

### Точечное профилирование в production

  

> [!warning] Осторожно

> Профилирование в production может существенно замедлить работу приложения.

  

```php

if (isset($_SESSION['user_id']) && in_array($_SESSION['user_id'], [1, 2, 3])) {

ini_set('xdebug.mode', 'profile');

ini_set('xdebug.start_with_request', 'trigger');

}

```

  

---

  

## Лучшие практики

  

### Безопасность

  

- Никогда не включайте xDebug в production без крайней необходимости

- Используйте `trigger`-режим вместо постоянной активации

- Ограничивайте доступ по IP-адресам

  

### Производительность

  

- Отключайте xDebug, когда не используете

- Настраивайте ограничения на глубину и размер данных

- Используйте профилирование точечно

  

### Разработка

  

- Настройте автоматическое включение в development-среде

- Используйте версионирование конфигурации

- Документируйте настройки для команды

  

### Отладка

  

- Используйте осмысленные имена для breakpoints

- Не забывайте удалять временные breakpoints

- Комбинируйте с логированием для комплексного анализа

  

---

  

## Заключение

  

xDebug — это неотъемлемый инструмент в арсенале PHP-разработчика. Он превращает процесс отладки из слепого поиска с помощью `var_dump()` и `echo` в управляемый, пошаговый процесс с полным контролем над выполнением кода.

  

Основные выводы:

  

- **Начните с `xdebug.mode=debug`** — этого достаточно для большинства задач пошаговой отладки

- **Используйте `trigger` вместо `yes`** — постоянная активация xDebug замедляет приложение в 2–10 раз

- **Настройте IDE один раз** — после правильной конфигурации PhpStorm или VS Code отладка работает прозрачно

- **Docker — не проблема** — `host.docker.internal` решает проблему подключения из контейнера

- **Профилирование — для узких мест** — не профилируйте всё подряд, используйте только для поиска конкретных проблем

  

Версия xDebug 3.x значительно упростила конфигурацию по сравнению с 2.x: один параметр `xdebug.mode` заменяет десятки отдельных настроек. Если вы обновляетесь — обязательно ознакомьтесь с [гайдом по миграции](https://xdebug.org/docs/upgrade_guide).

  

---

  

## Полезные ссылки

  

- [Официальный сайт xDebug](https://xdebug.org)

- [Документация xDebug 3](https://xdebug.org/docs)

- [Гайд по обновлению с xDebug 2 до 3](https://xdebug.org/docs/upgrade_guide)

- [Репозиторий xDebug на GitHub](https://github.com/xdebug/xdebug)

- [Страница установки (Wizard)](https://xdebug.org/wizard) — определяет нужную версию по выводу `phpinfo()`

- [Поддержка xDebug в PhpStorm](https://www.jetbrains.com/help/phpstorm/configuring-xdebug.html)

- [Расширение PHP Debug для VS Code](https://marketplace.visualstudio.com/items?itemName=xdebug.php-debug)

- [Протокол DBGP](https://xdebug.org/docs-dbgp.php) — спецификация протокола отладки

- [KCacheGrind](https://kcachegrind.github.io) — визуализатор результатов профилирования

- [Xdebug Helper (Chrome)](https://chromewebstore.google.com/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc) — расширение для браузера