# php.ini и директивы PHP

  

## Связанные заметки

  

- [[LAMP и LEMP]]

- [[Linux Управление процессами]]

- [[Linux Файловая система]]

- [[xDebug]]

  

---

  

## Что такое php.ini

  

**php.ini** — основной конфигурационный файл PHP, определяющий поведение интерпретатора во время выполнения скриптов. Содержит директивы (настройки), контролирующие различные аспекты работы PHP.

  

### Расположение php.ini

  

PHP ищет файл `php.ini` в следующих местах (в порядке приоритета):

  

**Linux/macOS:**

  

```bash

/etc/php/8.1/apache2/php.ini # для Apache

/etc/php/8.1/fpm/php.ini # для PHP-FPM

/etc/php/8.1/cli/php.ini # для CLI

/usr/local/etc/php/php.ini # установка через Homebrew

/opt/lampp/etc/php.ini # XAMPP

```

  

**Windows:**

  

```bash

C:\php\php.ini

C:\Windows\php.ini

C:\xampp\php\php.ini

```

  

### Определение текущего расположения php.ini

  

```php

<?php

echo php_ini_loaded_file();

echo php_ini_scanned_files();

phpinfo();

?>

```

  

```bash

php --ini

php -i | grep memory_limit

```

  

### Структура php.ini

  

Файл состоит из:

- **Комментариев** — начинаются с `;` или `#`

- **Секций** — в квадратных скобках `[section]`

- **Директив** — формат `directive = value`

  

```ini

; Это комментарий

[PHP]

engine = On

short_open_tag = Off

  

[Date]

date.timezone = "Europe/Moscow"

  

[Session]

session.save_handler = files

```

  

### Типы значений в php.ini

  

**Булевы значения:**

  

```ini

; Включено (все эквивалентны)

directive = On

directive = Yes

directive = 1

directive = True

  

; Выключено (все эквивалентны)

directive = Off

directive = No

directive = 0

directive = False

```

  

**Размеры (байты):**

  

```ini

memory_limit = 128M ; 128 мегабайт

upload_max_filesize = 2G ; 2 гигабайта

post_max_size = 8M ; 8 мегабайт

  

; Суффиксы: K/k = килобайты, M/m = мегабайты, G/g = гигабайты

```

  

**Время:**

  

```ini

max_execution_time = 30 ; секунды

max_input_time = 60 ; секунды

```

  

### Применение изменений

  

**Для веб-сервера:**

  

```bash

# Apache

sudo systemctl restart apache2

  

# Nginx + PHP-FPM

sudo systemctl restart nginx

sudo systemctl restart php8.1-fpm

```

  

**Для CLI:** изменения применяются немедленно при следующем запуске PHP-скрипта.

  

---

  

## mbstring (Multibyte String)

  

> [!info] Расширение mbstring

> **mbstring** — расширение PHP для работы с многобайтными строками. Критически важно для поддержки Unicode и различных кодировок, особенно при работе с кириллицей.

  

### Основные директивы mbstring

  

| Директива | Описание | Пример |

|-----------|----------|--------|

| `mbstring.language` | Язык по умолчанию | `"Russian"` |

| `mbstring.internal_encoding` | Внутренняя кодировка | `"UTF-8"` |

| `mbstring.http_input` | Кодировка входящих данных | `"auto"` |

| `mbstring.http_output` | Кодировка HTTP-вывода | `"UTF-8"` |

| `mbstring.encoding_translation` | Автоконвертация кодировок | `On` |

| `mbstring.detect_order` | Порядок определения кодировки | `"auto"` |

| `mbstring.substitute_character` | Символ замены для некорректных последовательностей | `"?"` |

  

### Практические примеры

  

```php

<?php

$text = "Привет мир!";

  

echo strlen($text); // 21 (байты)

echo mb_strlen($text); // 11 (символы)

  

echo mb_substr("Очень длинный текст", 0, 10); // "Очень длин"

  

// Конвертация кодировок

$windows = mb_convert_encoding($text, 'Windows-1251', 'UTF-8');

$utf8 = mb_convert_encoding($windows, 'UTF-8', 'Windows-1251');

  

// Определение кодировки

$enc = mb_detect_encoding($text, ['UTF-8', 'Windows-1251', 'ISO-8859-1']);

  

// Проверка корректности

if (mb_check_encoding($text, 'UTF-8')) {

echo "Корректная UTF-8";

}

?>

```

  

### Конфигурация для русского языка

  

```ini

mbstring.language = "Russian"

mbstring.internal_encoding = "UTF-8"

mbstring.http_input = "auto"

mbstring.http_output = "UTF-8"

mbstring.detect_order = "UTF-8,Windows-1251,ISO-8859-5"

```

  

---

  

## memory_limit

  

> [!tip] Правило памяти

> Память должна быть согласована: `memory_limit` > [[#post_max_size]] > [[#upload_max_filesize]]

  

Определяет максимальный объем памяти для PHP-скрипта.

  

### Рекомендуемые значения

  

| Тип приложения | Значение |

|----------------|----------|

| Простые сайты | `64M` |

| Стандартные приложения | `128M` |

| CMS (WordPress, Drupal) | `256M` |

| Обработка изображений / большие данные | `512M` |

| CLI-скрипты | `1G` |

  

```ini

memory_limit = 128M ; рекомендуемое

memory_limit = -1 ; без ограничений (НЕ рекомендуется в production)

```

  

### Динамическое изменение

  

```php

<?php

echo "Текущий лимит: " . ini_get('memory_limit') . "\n";

ini_set('memory_limit', '256M');

echo "Новый лимит: " . ini_get('memory_limit') . "\n";

  

echo "Использовано: " . memory_get_usage(true) . " байт\n";

echo "Пик: " . memory_get_peak_usage(true) . " байт\n";

?>

```

  

### Мониторинг использования памяти

  

```php

<?php

class MemoryMonitor {

private array $checkpoints = [];

  

public function checkpoint(string $name): void {

$this->checkpoints[$name] = [

'usage' => memory_get_usage(true),

'peak' => memory_get_peak_usage(true),

'time' => microtime(true)

];

}

  

public function report(): void {

foreach ($this->checkpoints as $name => $data) {

echo sprintf(

"%s: %s (пик: %s)\n",

$name,

$this->formatBytes($data['usage']),

$this->formatBytes($data['peak'])

);

}

}

  

private function formatBytes(int $size): string {

$units = ['B', 'KB', 'MB', 'GB'];

for ($i = 0; $size > 1024 && $i < count($units) - 1; $i++) {

$size /= 1024;

}

return round($size, 2) . ' ' . $units[$i];

}

}

?>

```

  

### Оптимизация памяти

  

> [!warning] Плохо vs Хорошо

> **Плохо:** `file_get_contents()` — загружает весь файл в память.

> **Хорошо:** построчная обработка через `fopen()`/`fgets()` или генераторы.

  

```php

<?php

// Генератор для построчного чтения большого файла

function readLargeFile(string $filename): Generator {

$handle = fopen($filename, 'r');

if ($handle) {

while (($line = fgets($handle)) !== false) {

yield trim($line);

}

fclose($handle);

}

}

  

foreach (readLargeFile('large_file.txt') as $line) {

// Обработка каждой строки с минимальным потреблением памяти

}

?>

```

  

---

  

## post_max_size

  

Определяет максимальный размер данных, отправляемых методом POST.

  

### Важные соотношения директив

  

```

memory_limit > post_max_size > upload_max_filesize

128M > 8M > 2M

```

  

```ini

upload_max_filesize = 2M ; макс. размер одного файла

post_max_size = 8M ; макс. размер всех POST-данных

max_file_uploads = 20 ; макс. количество файлов

memory_limit = 128M ; должен быть > post_max_size

```

  

### Обработка POST-загрузки

  

```php

<?php

function checkPostSize(): bool {

if (empty($_POST) && empty($_FILES) && $_SERVER['CONTENT_LENGTH'] > 0) {

echo "Ошибка: размер данных превышает post_max_size\n";

return false;

}

return true;

}

  

if (checkPostSize()) {

if (isset($_FILES['files'])) {

foreach ($_FILES['files']['name'] as $key => $name) {

if ($_FILES['files']['error'][$key] === UPLOAD_ERR_OK) {

$tmpName = $_FILES['files']['tmp_name'][$key];

move_uploaded_file($tmpName, "uploads/$name");

}

}

}

}

?>

```

  

### Отладка POST-данных

  

```php

<?php

function debugPostData(): void {

echo "post_max_size: " . ini_get('post_max_size') . "\n";

echo "upload_max_filesize: " . ini_get('upload_max_filesize') . "\n";

echo "CONTENT_LENGTH: " . $_SERVER['CONTENT_LENGTH'] . " байт\n";

  

if (empty($_POST) && empty($_FILES) && $_SERVER['CONTENT_LENGTH'] > 0) {

echo "ВНИМАНИЕ: данные не получены, превышен post_max_size\n";

}

}

?>

```

  

---

  

## short_open_tag

  

> [!danger] Не рекомендуется

> Короткие теги (`<? ... ?>`) зависят от настройки `short_open_tag = On` и вызывают конфликты с XML-декларациями. Всегда используйте полные теги `<?php ?>`.

  

### Конфигурация

  

```ini

short_open_tag = Off ; рекомендуется

short_open_tag = On ; НЕ рекомендуется

```

  

### Сравнение тегов

  

| Тег | Доступность | Рекомендация |

|-----|-------------|--------------|

| `<?php ... ?>` | Всегда | Да |

| `<?= $var ?>` | Всегда (PHP 5.4+) | Да |

| `<? ... ?>` | Только при `short_open_tag = On` | Нет |

  

### Проблема с XML

  

```xml

<?xml version="1.0" encoding="UTF-8"?>

<!-- Если short_open_tag = On, PHP попытается обработать это как PHP-код -->

```

  

### Рекомендуемый подход в шаблонах

  

```php

<?php foreach ($users as $user): ?>

<li><?= htmlspecialchars($user->name) ?></li>

<?php endforeach; ?>

```

  

### Миграция с коротких тегов

  

```bash

# macOS

find . -name "*.php" -exec sed -i '' 's/<? /<?php /g' {} \;

```

  

```php

<?php

// Программная замена

$content = preg_replace('/(<\?(?!php|=))/', '<?php', $content);

?>

```

  

---

  

## upload_max_filesize

  

Определяет максимальный размер файла, загружаемого через HTTP.

  

### Связанные директивы

  

```ini

file_uploads = On

upload_max_filesize = 10M

max_file_uploads = 20

post_max_size = 50M ; > upload_max_filesize

memory_limit = 128M ; > post_max_size

max_execution_time = 300 ; время для загрузки

max_input_time = 300 ; время парсинга

```

  

### Обработка загрузки файлов

  

```php

<?php

function handleFileUpload(string $fileInputName): array {

if (!isset($_FILES[$fileInputName])) {

return ['error' => 'Файл не найден'];

}

  

$file = $_FILES[$fileInputName];

  

switch ($file['error']) {

case UPLOAD_ERR_OK: break;

case UPLOAD_ERR_INI_SIZE: return ['error' => 'Превышен upload_max_filesize'];

case UPLOAD_ERR_FORM_SIZE: return ['error' => 'Превышен MAX_FILE_SIZE'];

case UPLOAD_ERR_PARTIAL: return ['error' => 'Файл загружен частично'];

case UPLOAD_ERR_NO_FILE: return ['error' => 'Файл не загружен'];

case UPLOAD_ERR_NO_TMP_DIR: return ['error' => 'Нет временной папки'];

case UPLOAD_ERR_CANT_WRITE: return ['error' => 'Ошибка записи'];

case UPLOAD_ERR_EXTENSION: return ['error' => 'Загрузка остановлена расширением'];

default: return ['error' => 'Неизвестная ошибка'];

}

  

$allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];

if (!in_array($file['type'], $allowedTypes)) {

return ['error' => 'Недопустимый тип файла'];

}

  

$filename = preg_replace('/[^a-zA-Z0-9._-]/', '_', $file['name']);

$name = pathinfo($filename, PATHINFO_FILENAME);

$ext = pathinfo($filename, PATHINFO_EXTENSION);

$filename = $name . '_' . time() . '.' . $ext;

  

if (move_uploaded_file($file['tmp_name'], 'uploads/' . $filename)) {

return ['success' => true, 'filename' => $filename];

}

return ['error' => 'Ошибка сохранения'];

}

?>

```

  

### Chunked upload (загрузка большими файлами)

  

```javascript

class ChunkedUploader {

constructor(file, chunkSize = 1024 * 1024) {

this.file = file;

this.chunkSize = chunkSize;

this.totalChunks = Math.ceil(file.size / chunkSize);

this.currentChunk = 0;

}

  

async upload() {

while (this.currentChunk < this.totalChunks) {

const start = this.currentChunk * this.chunkSize;

const end = Math.min(start + this.chunkSize, this.file.size);

const chunk = this.file.slice(start, end);

  

const formData = new FormData();

formData.append('chunk', chunk);

formData.append('chunkIndex', this.currentChunk);

formData.append('totalChunks', this.totalChunks);

formData.append('fileName', this.file.name);

  

await fetch('upload_chunk.php', { method: 'POST', body: formData });

this.currentChunk++;

}

}

}

```

  

```php

<?php

// upload_chunk.php

$chunkIndex = $_POST['chunkIndex'];

$totalChunks = $_POST['totalChunks'];

$fileName = $_POST['fileName'];

$tempDir = sys_get_temp_dir() . '/uploads/';

$chunkFile = $tempDir . $fileName . '.part' . $chunkIndex;

  

move_uploaded_file($_FILES['chunk']['tmp_name'], $chunkFile);

  

if ($chunkIndex == $totalChunks - 1) {

$output = fopen('uploads/' . $fileName, 'wb');

for ($i = 0; $i < $totalChunks; $i++) {

$input = fopen($tempDir . $fileName . '.part' . $i, 'rb');

stream_copy_to_stream($input, $output);

fclose($input);

unlink($tempDir . $fileName . '.part' . $i);

}

fclose($output);

}

?>

```

  

---

  

## max_execution_time

  

Определяет максимальное время выполнения PHP-скрипта в секундах.

  

> [!note] CLI vs Web

> В CLI-режиме `max_execution_time = 0` (без ограничений) по умолчанию. В веб-режиме ограничение активно.

  

### Конфигурация

  

```ini

max_execution_time = 30 ; по умолчанию

max_execution_time = 300 ; для загрузки файлов

max_execution_time = 0 ; без ограничений (только CLI)

```

  

### Динамическое управление временем

  

```php

<?php

set_time_limit(300); // Увеличить до 5 минут

  

function getRemainingTime() {

$startTime = $_SERVER['REQUEST_TIME_FLOAT'] ?? microtime(true);

$maxTime = ini_get('max_execution_time');

$elapsed = microtime(true) - $startTime;

return $maxTime == 0 ? 'unlimited' : max(0, $maxTime - $elapsed);

}

?>

```

  

### Мониторинг производительности

  

```php

<?php

class PerformanceMonitor {

private float $startTime;

private array $checkpoints = [];

  

public function __construct() {

$this->startTime = microtime(true);

$this->checkpoint('start');

}

  

public function checkpoint(string $name): void {

$current = microtime(true);

$this->checkpoints[$name] = [

'elapsed' => $current - $this->startTime,

'memory' => memory_get_usage(true),

];

}

  

public function getReport(): string {

$maxTime = ini_get('max_execution_time');

$report = "Макс. время: $maxTime сек\n";

  

foreach ($this->checkpoints as $name => $data) {

$report .= sprintf("%s: %.4f сек\n", $name, $data['elapsed']);

}

return $report;

}

}

?>

```

  

---

  

## Заключение

  

### Иерархия директив

  

```

memory_limit (128M)

└── post_max_size (8M)

└── upload_max_filesize (2M)

```

  

> [!important] Главное правило

> Каждая верхнеуровневая директива должна быть больше вложенной. Нарушение этого правила приведёт к немым ошибкам при загрузке файлов.

  

### Чеклист настройки php.ini

  

- [ ] `memory_limit` достаточно для приложения

- [ ] `post_max_size` > `upload_max_filesize`

- [ ] `memory_limit` > `post_max_size`

- [ ] `max_execution_time` учитывает длительные операции

- [ ] `mbstring` настроен для вашей кодировки (`UTF-8`)

- [ ] `short_open_tag = Off` для совместимости

- [ ] `file_uploads = On` если нужна загрузка файлов

- [ ] Перезапуск веб-сервера после изменений

  

### Быстрая шпаргалка

  

| Директива | По умолчанию | Рекомендация | См. также |

|-----------|-------------|--------------|-----------|

| `memory_limit` | `128M` | `128M`–`256M` | [[#memory_limit]] |

| `post_max_size` | `8M` | `> upload_max_filesize` | [[#post_max_size]] |

| `upload_max_filesize` | `2M` | по потребности | [[#upload_max_filesize]] |

| `max_execution_time` | `30` | `30`–`300` | [[#max_execution_time]] |

| `short_open_tag` | `Off` | `Off` | [[#short_open_tag]] |

| `mbstring.language` | `English` | по языку проекта | [[#mbstring (Multibyte String)]] |

  

---

  

**См. также:** [[LAMP и LEMP]] · [[Linux Управление процессами]] · [[Linux Файловая система]] · [[xDebug]]