
**## Что такое php.ini**

  

**php.ini** - это основной конфигурационный файл PHP, который определяет поведение интерпретатора PHP во время выполнения скриптов. Этот файл содержит множество директив (настроек), которые контролируют различные аспекты работы PHP.

  

**### Расположение php.ini**

  

PHP ищет файл php.ini в следующих местах (в порядке приоритета):

  

#### Linux/macOS:

```

/etc/php/8.1/apache2/php.ini  (для Apache)

/etc/php/8.1/fpm/php.ini      (для PHP-FPM)

/etc/php/8.1/cli/php.ini      (для CLI)

/usr/local/etc/php/php.ini    (установка через Homebrew)

/opt/lampp/etc/php.ini        (XAMPP)

```

  

#### Windows:

```

C:\php\php.ini

C:\Windows\php.ini

C:\xampp\php\php.ini

```

  

**### Определение текущего расположения php.ini**

  

```php

<?php

// Показать путь к активному php.ini

echo php_ini_loaded_file();

  

// Показать все конфигурационные файлы

echo php_ini_scanned_files();

  

// Показать информацию о PHP

phpinfo();

?>

```

  

```bash

# Через командную строку

php --ini

  

# Показать конкретную директиву

php -i | grep memory_limit

```

  

**### Структура php.ini**

  

Файл php.ini состоит из:

- **Комментариев** (начинаются с ; или #)

- **Секций** в квадратных скобках [section]

- **Директив** в формате `directive = value`

  

```ini

; Это комментарий

[PHP]

; Основные настройки PHP

engine = On

short_open_tag = Off

  

[Date]

; Настройки даты и времени

date.timezone = "Europe/Moscow"

  

[Session]

; Настройки сессий

session.save_handler = files

```

  

**### Типы значений в php.ini**

  

#### Булевы значения:

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

  

#### Размеры (байты):

```ini

; Различные форматы указания размера

memory_limit = 128M      ; 128 мегабайт

upload_max_filesize = 2G ; 2 гигабайта

post_max_size = 8M       ; 8 мегабайт

  

; Суффиксы:

; K или k = килобайты

; M или m = мегабайты  

; G или g = гигабайты

```

  

#### Время:

```ini

max_execution_time = 30  ; секунды

max_input_time = 60      ; секунды

```

  

**### Применение изменений**

  

После изменения php.ini необходимо:

  

#### Для веб-сервера:

```bash

# Apache

sudo systemctl restart apache2

# или

sudo service apache2 restart

  

# Nginx с PHP-FPM

sudo systemctl restart nginx

sudo systemctl restart php8.1-fpm

  

# Windows (XAMPP)

# Перезапустить Apache через панель управления XAMPP

```

  

#### Для CLI:

Изменения применяются немедленно при следующем запуске PHP-скрипта.

  

---

  

**## mbstring (Multibyte String)**

  

**mbstring** - это расширение PHP для работы с многобайтными строками, особенно важное для поддержки Unicode и различных кодировок текста.

  

**### Основные директивы mbstring**

  

#### mbstring.language

```ini

; Язык по умолчанию для mbstring функций

mbstring.language = "Russian"

; Возможные значения: English, Japanese, Korean, Simplified Chinese, Traditional Chinese, Russian, etc.

```

  

#### mbstring.internal_encoding

```ini

; Внутренняя кодировка для mbstring функций

mbstring.internal_encoding = "UTF-8"

; Рекомендуется всегда использовать UTF-8

```

  

#### mbstring.http_input

```ini

; Автоматическое определение кодировки входящих данных

mbstring.http_input = "auto"

; Возможные значения: auto, pass, UTF-8, EUC-JP, SJIS, etc.

```

  

#### mbstring.http_output

```ini

; Кодировка для вывода HTTP

mbstring.http_output = "UTF-8"

```

  

#### mbstring.encoding_translation

```ini

; Включение автоматической конвертации кодировок

mbstring.encoding_translation = On

```

  

#### mbstring.detect_order

```ini

; Порядок определения кодировки

mbstring.detect_order = "auto"

; Или конкретный список: "UTF-8,Windows-1251,ISO-8859-1"

```

  

#### mbstring.substitute_character

```ini

; Символ замены для недопустимых последовательностей

mbstring.substitute_character = "?"

; Или Unicode код: "U+003F"

```

  

**### Практические примеры использования mbstring**

  

```php

<?php

// Проверка длины многобайтной строки

$text = "Привет мир!";

echo strlen($text);    // 21 (количество байт)

echo mb_strlen($text); // 11 (количество символов)

  

// Обрезка многобайтной строки

$long_text = "Очень длинный текст на русском языке";

echo mb_substr($long_text, 0, 10); // "Очень длин"

  

// Конвертация кодировок

$windows_text = mb_convert_encoding($text, 'Windows-1251', 'UTF-8');

$utf8_text = mb_convert_encoding($windows_text, 'UTF-8', 'Windows-1251');

  

// Определение кодировки

$encoding = mb_detect_encoding($text, ['UTF-8', 'Windows-1251', 'ISO-8859-1']);

echo $encoding; // UTF-8

  

// Проверка корректности кодировки

if (mb_check_encoding($text, 'UTF-8')) {

    echo "Текст в корректной UTF-8 кодировке";

}

?>

```

  

**### Конфигурация для разных языков**

  

#### Русский язык:

```ini

mbstring.language = "Russian"

mbstring.internal_encoding = "UTF-8"

mbstring.http_input = "auto"

mbstring.http_output = "UTF-8"

mbstring.detect_order = "UTF-8,Windows-1251,ISO-8859-5"

```

  

#### Японский язык:

```ini

mbstring.language = "Japanese"

mbstring.internal_encoding = "UTF-8"

mbstring.http_input = "auto"

mbstring.http_output = "UTF-8"

mbstring.detect_order = "UTF-8,EUC-JP,SJIS,JIS"

```

  

---

  

**## memory_limit**

  

**memory_limit** определяет максимальный объем памяти, который может использовать PHP-скрипт.

  

**### Синтаксис и значения**

  

```ini

; Различные способы указания лимита памяти

memory_limit = 128M      ; 128 мегабайт (рекомендуемое значение)

memory_limit = 256M      ; 256 мегабайт

memory_limit = 512M      ; 512 мегабайт

memory_limit = 1G        ; 1 гигабайт

memory_limit = -1        ; Без ограничений (НЕ рекомендуется в production)

```

  

**### Рекомендуемые значения**

  

#### Типы приложений:

```ini

; Простые сайты

memory_limit = 64M

  

; Стандартные веб-приложения

memory_limit = 128M

  

; CMS (WordPress, Drupal)

memory_limit = 256M

  

; Тяжелые приложения (обработка изображений, большие данные)

memory_limit = 512M

  

; CLI скрипты для обработки данных

memory_limit = 1G

  

; Разработка и отладка

memory_limit = 256M

```

  

**### Динамическое изменение memory_limit**

  

```php

<?php

// Проверка текущего лимита

echo "Текущий лимит: " . ini_get('memory_limit') . "\n";

  

// Изменение лимита во время выполнения

ini_set('memory_limit', '256M');

  

// Проверка измененного лимита

echo "Новый лимит: " . ini_get('memory_limit') . "\n";

  

// Получение использования памяти

echo "Использовано памяти: " . memory_get_usage(true) . " байт\n";

echo "Пиковое использование: " . memory_get_peak_usage(true) . " байт\n";

  

// Форматированный вывод

function formatBytes($size, $precision = 2) {

    $units = ['B', 'KB', 'MB', 'GB'];

    for ($i = 0; $size > 1024 && $i < count($units) - 1; $i++) {

        $size /= 1024;

    }

    return round($size, $precision) . ' ' . $units[$i];

}

  

echo "Использовано: " . formatBytes(memory_get_usage(true)) . "\n";

?>

```

  

**### Мониторинг использования памяти**

  

```php

<?php

class MemoryMonitor {

    private $checkpoints = [];

    public function checkpoint($name) {

        $this->checkpoints[$name] = [

            'usage' => memory_get_usage(true),

            'peak' => memory_get_peak_usage(true),

            'time' => microtime(true)

        ];

    }

    public function report() {

        foreach ($this->checkpoints as $name => $data) {

            echo sprintf(

                "%s: %s (пик: %s)\n",

                $name,

                $this->formatBytes($data['usage']),

                $this->formatBytes($data['peak'])

            );

        }

    }

    private function formatBytes($size) {

        $units = ['B', 'KB', 'MB', 'GB'];

        for ($i = 0; $size > 1024 && $i < count($units) - 1; $i++) {

            $size /= 1024;

        }

        return round($size, 2) . ' ' . $units[$i];

    }

}

  

$monitor = new MemoryMonitor();

$monitor->checkpoint('Начало скрипта');

  

// Создание большого массива

$data = array_fill(0, 100000, 'test data');

$monitor->checkpoint('После создания массива');

  

// Обработка данных

foreach ($data as &$item) {

    $item = strtoupper($item);

}

$monitor->checkpoint('После обработки данных');

  

$monitor->report();

?>

```

  

**### Оптимизация использования памяти**

  

```php

<?php

// Плохой пример - загрузка всего файла в память

function badFileProcessing($filename) {

    $content = file_get_contents($filename); // Загружает весь файл

    $lines = explode("\n", $content);

    return count($lines);

}

  

// Хороший пример - построчная обработка

function goodFileProcessing($filename) {

    $count = 0;

    $handle = fopen($filename, 'r');

    if ($handle) {

        while (($line = fgets($handle)) !== false) {

            $count++;

        }

        fclose($handle);

    }

    return $count;

}

  

// Использование генераторов для экономии памяти

function readLargeFile($filename) {

    $handle = fopen($filename, 'r');

    if ($handle) {

        while (($line = fgets($handle)) !== false) {

            yield trim($line);

        }

        fclose($handle);

    }

}

  

// Обработка большого файла с минимальным использованием памяти

foreach (readLargeFile('large_file.txt') as $line) {

    // Обработка каждой строки

    echo $line . "\n";

}

?>

```

  

---

  

**## post_max_size**

  

**post_max_size** определяет максимальный размер данных, которые могут быть отправлены методом POST.

  

**### Конфигурация и значения**

  

```ini

; Различные варианты настройки

post_max_size = 8M       ; По умолчанию (8 мегабайт)

post_max_size = 16M      ; Для форм с файлами

post_max_size = 32M      ; Для загрузки больших файлов

post_max_size = 100M     ; Для файловых хранилищ

post_max_size = 0        ; Без ограничений (НЕ рекомендуется)

```

  

**### Связь с другими директивами**

  

```ini

; Важное правило: post_max_size должен быть больше upload_max_filesize

upload_max_filesize = 2M    ; Максимальный размер одного файла

post_max_size = 8M          ; Максимальный размер всех POST данных

  

; Также влияет на:

max_file_uploads = 20       ; Максимальное количество файлов

memory_limit = 128M         ; Должен быть больше post_max_size

```

  

**### Практические примеры**

  

#### Форма загрузки файлов:

```html

<!-- HTML форма -->

<form action="upload.php" method="post" enctype="multipart/form-data">

    <input type="file" name="files[]" multiple>

    <input type="text" name="description" maxlength="1000">

    <input type="submit" value="Загрузить">

</form>

```

  

```php

<?php

// upload.php - обработка загрузки

function checkPostSize() {

    $postMaxSize = ini_get('post_max_size');

    $uploadMaxSize = ini_get('upload_max_filesize');

    echo "Максимальный размер POST: $postMaxSize\n";

    echo "Максимальный размер файла: $uploadMaxSize\n";

    echo "Размер полученных данных: " . $_SERVER['CONTENT_LENGTH'] . " байт\n";

    // Проверка превышения лимита

    if (empty($_POST) && empty($_FILES) && $_SERVER['CONTENT_LENGTH'] > 0) {

        echo "Ошибка: размер данных превышает post_max_size\n";

        return false;

    }

    return true;

}

  

if (checkPostSize()) {

    // Обработка загруженных файлов

    if (isset($_FILES['files'])) {

        foreach ($_FILES['files']['name'] as $key => $name) {

            if ($_FILES['files']['error'][$key] === UPLOAD_ERR_OK) {

                $tmpName = $_FILES['files']['tmp_name'][$key];

                $size = $_FILES['files']['size'][$key];

                echo "Загружен файл: $name (размер: $size байт)\n";

                // Перемещение файла

                move_uploaded_file($tmpName, "uploads/$name");

            } else {

                echo "Ошибка загрузки файла: $name\n";

            }

        }

    }

    // Обработка текстовых данных

    if (isset($_POST['description'])) {

        echo "Описание: " . $_POST['description'] . "\n";

    }

}

?>

```

  

#### AJAX загрузка с проверкой размера:

```javascript

// Проверка размера на клиенте

function validateFileSize(input) {

    const maxSize = <?php echo ini_get('post_max_size'); ?> * 1024 * 1024; // Конвертация в байты

    let totalSize = 0;

    for (let file of input.files) {

        totalSize += file.size;

    }

    if (totalSize > maxSize) {

        alert('Общий размер файлов превышает максимально допустимый');

        return false;

    }

    return true;

}

  

// AJAX загрузка с индикатором прогресса

function uploadFiles(formData) {

    const xhr = new XMLHttpRequest();

    xhr.upload.addEventListener('progress', function(e) {

        if (e.lengthComputable) {

            const percentage = (e.loaded / e.total) * 100;

            document.getElementById('progress').style.width = percentage + '%';

        }

    });

    xhr.onreadystatechange = function() {

        if (xhr.readyState === 4) {

            if (xhr.status === 200) {

                console.log('Файлы загружены успешно');

            } else {

                console.log('Ошибка загрузки');

            }

        }

    };

    xhr.open('POST', 'upload.php');

    xhr.send(formData);

}

```

  

**### Мониторинг и отладка**

  

```php

<?php

function debugPostData() {

    echo "=== Информация о POST данных ===\n";

    echo "Размер полученных данных: " . $_SERVER['CONTENT_LENGTH'] . " байт\n";

    echo "post_max_size: " . ini_get('post_max_size') . "\n";

    echo "upload_max_filesize: " . ini_get('upload_max_filesize') . "\n";

    echo "max_file_uploads: " . ini_get('max_file_uploads') . "\n";

    echo "memory_limit: " . ini_get('memory_limit') . "\n";

    echo "\n=== POST переменные ===\n";

    echo "Количество POST переменных: " . count($_POST) . "\n";

    foreach ($_POST as $key => $value) {

        echo "$key: " . strlen($value) . " символов\n";

    }

    echo "\n=== Загруженные файлы ===\n";

    echo "Количество файлов: " . count($_FILES) . "\n";

    foreach ($_FILES as $key => $file) {

        if (is_array($file['name'])) {

            echo "$key: " . count($file['name']) . " файлов\n";

        } else {

            echo "$key: {$file['name']} ({$file['size']} байт)\n";

        }

    }

    // Проверка ошибок

    if (empty($_POST) && empty($_FILES) && $_SERVER['CONTENT_LENGTH'] > 0) {

        echo "\n⚠️ ВНИМАНИЕ: Данные не получены, возможно превышен post_max_size\n";

    }

}

  

debugPostData();

?>

```

  

---

  

**## short_open_tag**

  

**short_open_tag** определяет, разрешено ли использование сокращенного синтаксиса открытия PHP тегов.

  

**### Конфигурация**

  

```ini

; Включение коротких тегов (НЕ рекомендуется)

short_open_tag = On

  

; Отключение коротких тегов (рекомендуется)

short_open_tag = Off

```

  

**### Синтаксис и примеры**

  

#### Стандартные теги (рекомендуется):

```php

<?php

echo "Привет мир!";

?>

```

  

#### Короткие теги (зависят от short_open_tag):

```php

<?

echo "Привет мир!";

?>

  

<!-- Короткий echo тег -->

<p>Имя пользователя: <?= $username ?></p>

```

  

#### Теги для вывода (всегда доступны с PHP 5.4+):

```php

<!-- Эти теги работают независимо от short_open_tag -->

<p>Дата: <?= date('Y-m-d') ?></p>

<p>Время: <?= time() ?></p>

```

  

**### Проблемы с короткими тегами**

  

#### Конфликт с XML:

```xml

<?xml version="1.0" encoding="UTF-8"?>

<root>

    <!-- Если short_open_tag = On, это может вызвать ошибку -->

</root>

```

  

#### Проблемы переносимости:

```php

<?php

// Этот код работает везде

echo "Надежный код";

?>

  

<?

// Этот код работает только если short_open_tag = On

echo "Ненадежный код";

?>

```

  

**### Рекомендации по использованию**

  

#### Правильный подход:

```php

<?php

// Всегда использовать полные теги

class UserController {

    public function index() {

        $users = $this->getUsers();

        return view('users.index', compact('users'));

    }

}

  

// Для вывода переменных в шаблонах

?>

<html>

<head>

    <title><?= htmlspecialchars($title) ?></title>

</head>

<body>

    <h1><?= htmlspecialchars($heading) ?></h1>

    <ul>

        <?php foreach ($users as $user): ?>

            <li><?= htmlspecialchars($user->name) ?></li>

        <?php endforeach; ?>

    </ul>

</body>

</html>

```

  

#### Проверка настройки в коде:

```php

<?php

function checkShortOpenTag() {

    $setting = ini_get('short_open_tag');

    if ($setting) {

        echo "⚠️ Короткие теги включены\n";

        echo "Рекомендуется отключить для лучшей совместимости\n";

    } else {

        echo "✅ Короткие теги отключены (рекомендуемая настройка)\n";

    }

    return (bool) $setting;

}

  

// Альтернативный способ проверки

if (ini_get('short_open_tag')) {

    // Код для случая, когда короткие теги включены

} else {

    // Код для случая, когда короткие теги отключены

}

?>

```

  

**### Миграция с коротких тегов**

  

#### Автоматическая замена:

```bash

# Замена коротких тегов в файлах

find . -name "*.php" -exec sed -i 's/<? /<?php /g' {} \;

find . -name "*.php" -exec sed -i 's/<?$/<?php/g' {} \;

  

# Для macOS (BSD sed)

find . -name "*.php" -exec sed -i '' 's/<? /<?php /g' {} \;

```

  

#### PHP скрипт для миграции:

```php

<?php

function convertShortTags($directory) {

    $iterator = new RecursiveIteratorIterator(

        new RecursiveDirectoryIterator($directory)

    );

    foreach ($iterator as $file) {

        if ($file->getExtension() === 'php') {

            $content = file_get_contents($file->getPathname());

            // Замена коротких тегов

            $content = preg_replace('/(<\?(?!php|=))/', '<?php', $content);

            file_put_contents($file->getPathname(), $content);

            echo "Обработан файл: " . $file->getPathname() . "\n";

        }

    }

}

  

convertShortTags('./src');

?>

```

  

---

  

**## upload_max_filesize**

  

**upload_max_filesize** определяет максимальный размер файла, который может быть загружен через HTTP.

  

**### Конфигурация**

  

```ini

; Различные варианты настройки

upload_max_filesize = 2M     ; По умолчанию (2 мегабайта)

upload_max_filesize = 10M    ; Для изображений

upload_max_filesize = 50M    ; Для документов

upload_max_filesize = 100M   ; Для видео файлов

upload_max_filesize = 1G     ; Для больших файлов

```

  

**### Связанные директивы**

  

```ini

; Все эти директивы должны быть согласованы

file_uploads = On                    ; Разрешение загрузки файлов

upload_max_filesize = 10M           ; Максимальный размер одного файла

max_file_uploads = 20               ; Максимальное количество файлов

post_max_size = 50M                 ; Должен быть больше upload_max_filesize

memory_limit = 128M                 ; Должен быть больше post_max_size

max_execution_time = 300            ; Время для загрузки больших файлов

max_input_time = 300                ; Время парсинга входных данных

```

  

**### Обработка загружаемых файлов**

  

#### Базовая обработка:

```php

<?php

function handleFileUpload($fileInputName) {

    // Проверка наличия файла

    if (!isset($_FILES[$fileInputName])) {

        return ['error' => 'Файл не найден'];

    }

    $file = $_FILES[$fileInputName];

    // Проверка ошибок загрузки

    switch ($file['error']) {

        case UPLOAD_ERR_OK:

            break;

        case UPLOAD_ERR_INI_SIZE:

            return ['error' => 'Файл превышает upload_max_filesize'];

        case UPLOAD_ERR_FORM_SIZE:

            return ['error' => 'Файл превышает MAX_FILE_SIZE'];

        case UPLOAD_ERR_PARTIAL:

            return ['error' => 'Файл загружен частично'];

        case UPLOAD_ERR_NO_FILE:

            return ['error' => 'Файл не был загружен'];

        case UPLOAD_ERR_NO_TMP_DIR:

            return ['error' => 'Отсутствует временная папка'];

        case UPLOAD_ERR_CANT_WRITE:

            return ['error' => 'Ошибка записи на диск'];

        case UPLOAD_ERR_EXTENSION:

            return ['error' => 'Загрузка остановлена расширением'];

        default:

            return ['error' => 'Неизвестная ошибка'];

    }

    // Дополнительные проверки

    $maxSize = ini_get('upload_max_filesize');

    $maxSizeBytes = convertToBytes($maxSize);

    if ($file['size'] > $maxSizeBytes) {

        return ['error' => "Файл слишком большой (макс. $maxSize)"];

    }

    // Проверка типа файла

    $allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];

    if (!in_array($file['type'], $allowedTypes)) {

        return ['error' => 'Недопустимый тип файла'];

    }

    // Безопасное имя файла

    $filename = sanitizeFilename($file['name']);

    $uploadPath = 'uploads/' . $filename;

    // Перемещение файла

    if (move_uploaded_file($file['tmp_name'], $uploadPath)) {

        return [

            'success' => true,

            'filename' => $filename,

            'size' => $file['size'],

            'path' => $uploadPath

        ];

    } else {

        return ['error' => 'Ошибка сохранения файла'];

    }

}

  

function convertToBytes($value) {

    $unit = strtolower(substr($value, -1));

    $value = (int) $value;

    switch ($unit) {

        case 'g': $value *= 1024;

        case 'm': $value *= 1024;

        case 'k': $value *= 1024;

    }

    return $value;

}

  

function sanitizeFilename($filename) {

    // Удаление опасных символов

    $filename = preg_replace('/[^a-zA-Z0-9._-]/', '_', $filename);

    // Предотвращение двойных точек

    $filename = preg_replace('/\.+/', '.', $filename);

    // Добавление временной метки для уникальности

    $extension = pathinfo($filename, PATHINFO_EXTENSION);

    $name = pathinfo($filename, PATHINFO_FILENAME);

    return $name . '_' . time() . '.' . $extension;

}

  

// Использование

$result = handleFileUpload('userfile');

if (isset($result['error'])) {

    echo "Ошибка: " . $result['error'];

} else {

    echo "Файл загружен: " . $result['filename'];

}

?>

```

  

#### Множественная загрузка файлов:

```php

<?php

function handleMultipleFileUpload($fileInputName) {

    if (!isset($_FILES[$fileInputName])) {

        return ['error' => 'Файлы не найдены'];

    }

    $files = $_FILES[$fileInputName];

    $results = [];

    // Нормализация структуры массива

    if (!is_array($files['name'])) {

        $files = [

            'name' => [$files['name']],

            'type' => [$files['type']],

            'tmp_name' => [$files['tmp_name']],

            'error' => [$files['error']],

            'size' => [$files['size']]

        ];

    }

    $maxFiles = ini_get('max_file_uploads');

    $fileCount = count($files['name']);

    if ($fileCount > $maxFiles) {

        return ['error' => "Слишком много файлов (макс. $maxFiles)"];

    }

    for ($i = 0; $i < $fileCount; $i++) {

        $file = [

            'name' => $files['name'][$i],

            'type' => $files['type'][$i],

            'tmp_name' => $files['tmp_name'][$i],

            'error' => $files['error'][$i],

            'size' => $files['size'][$i]

        ];

        $result = handleSingleFile($file);

        $results[] = $result;

    }

    return $results;

}

  

function handleSingleFile($file) {

    // Логика обработки одного файла (как в предыдущем примере)

    // ...

}

?>

```

  

**### Прогрессивная загрузка больших файлов**

  

#### Chunked upload:

```javascript

// JavaScript для разбивки файла на части

class ChunkedUploader {

    constructor(file, chunkSize = 1024 * 1024) { // 1MB chunks

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

            try {

                await this.uploadChunk(formData);

                this.currentChunk++;

                this.updateProgress();

            } catch (error) {

                console.error('Ошибка загрузки части файла:', error);

                break;

            }

        }

    }

    uploadChunk(formData) {

        return fetch('upload_chunk.php', {

            method: 'POST',

            body: formData

        });

    }

    updateProgress() {

        const progress = (this.currentChunk / this.totalChunks) * 100;

        document.getElementById('progress').style.width = progress + '%';

    }

}

```

  

```php

<?php

// upload_chunk.php - серверная часть для chunked upload

function handleChunkedUpload() {

    $chunkIndex = $_POST['chunkIndex'];

    $totalChunks = $_POST['totalChunks'];

    $fileName = $_POST['fileName'];

    $tempDir = sys_get_temp_dir() . '/uploads/';

    if (!is_dir($tempDir)) {

        mkdir($tempDir, 0755, true);

    }

    $chunkFile = $tempDir . $fileName . '.part' . $chunkIndex;

    // Сохранение части файла

    if (move_uploaded_file($_FILES['chunk']['tmp_name'], $chunkFile)) {

        // Проверка, все ли части загружены

        if ($chunkIndex == $totalChunks - 1) {

            // Объединение всех частей

            $finalFile = 'uploads/' . $fileName;

            $output = fopen($finalFile, 'wb');

            for ($i = 0; $i < $totalChunks; $i++) {

                $partFile = $tempDir . $fileName . '.part' . $i;

                $input = fopen($partFile, 'rb');

                stream_copy_to_stream($input, $output);

                fclose($input);

                unlink($partFile); // Удаление временного файла

            }

            fclose($output);

            return ['success' => true, 'file' => $finalFile];

        }

        return ['success' => true, 'chunk' => $chunkIndex];

    } else {

        return ['error' => 'Ошибка сохранения части файла'];

    }

}

  

echo json_encode(handleChunkedUpload());

?>

```

  

---

  

**## max_execution_time**

  

**max_execution_time** определяет максимальное время выполнения PHP-скрипта в секундах.

  

**### Конфигурация**

  

```ini

; Различные варианты настройки

max_execution_time = 30      ; По умолчанию (30 секунд)

max_execution_time = 60      ; Для более сложных операций

max_execution_time = 300     ; Для загрузки файлов

max_execution_time = 600     ; Для обработки данных

max_execution_time = 0       ; Без ограничений (только для CLI)

```

  

**### Особенности применения**

  

#### Веб-скрипты vs CLI:

```php

<?php

// Проверка режима выполнения

if (php_sapi_name() === 'cli') {

    // В CLI режиме max_execution_time = 0 по умолчанию

    echo "Выполняется в командной строке (без ограничения времени)\n";

} else {

    // В веб-режиме действует ограничение

    echo "Выполняется через веб-сервер\n";

    echo "Максимальное время: " . ini_get('max_execution_time') . " сек\n";

}

  

// Получение оставшегося времени

function getRemainingTime() {

    $startTime = $_SERVER['REQUEST_TIME_FLOAT'] ?? microtime(true);

    $maxTime = ini_get('max_execution_time');

    $elapsed = microtime(true) - $startTime;

    if ($maxTime == 0) {

        return 'unlimited';

    }

    return max(0, $maxTime - $elapsed);

}

  

echo "Оставшееся время: " . getRemainingTime() . " сек\n";

?>

```

  

**### Динамическое изменение времени выполнения**

  

```php

<?php

// Увеличение времени выполнения для конкретной операции

function processLargeDataset($data) {

    // Сохранение текущего значения

    $originalTime = ini_get('max_execution_time');

    // Увеличение времени на 5 минут

    set_time_limit(300);

    try {

        foreach ($data as $item) {

            // Длительная обработка каждого элемента

            processItem($item);

            // Проверка времени каждые 100 итераций

            if (count($processed) % 100 === 0) {

                $remaining = getRemainingTime();

                if ($remaining < 30) { // Меньше 30 секунд осталось

                    echo "Мало времени, останавливаем обработку\n";

                    break;

                }

            }

        }

    } finally {

        // Восстановление исходного значения

        set_time_limit($originalTime);

    }

}

  

// Сброс таймера (начать отсчет заново)

function resetExecutionTimer() {

    set_time_limit(ini_get('max_execution_time'));

}

  

// Пример длительной операции с управлением временем

function importDataFromFile($filename) {

    $startTime = microtime(true);

    $processed = 0;

    $errors = 0;

    // Установка времени выполнения на 10 минут

    set_time_limit(600);

    $handle = fopen($filename, 'r');

    if (!$handle) {

        throw new Exception("Не удалось открыть файл: $filename");

    }

    try {

        while (($line = fgets($handle)) !== false) {

            try {

                processDataLine($line);

                $processed++;

                // Каждые 1000 записей проверяем время

                if ($processed % 1000 === 0) {

                    $elapsed = microtime(true) - $startTime;

                    $remaining = ini_get('max_execution_time') - $elapsed;

                    echo "Обработано: $processed записей\n";

                    echo "Прошло времени: " . round($elapsed, 2) . " сек\n";

                    echo "Осталось времени: " . round($remaining, 2) . " сек\n";

                    if ($remaining < 60) { // Меньше минуты

                        echo "Мало времени, сохраняем прогресс...\n";

                        saveProgress($processed);

                        break;

                    }

                    // Сброс таймера для продолжения работы

                    set_time_limit(600);

                }

            } catch (Exception $e) {

                $errors++;

                error_log("Ошибка обработки строки $processed: " . $e->getMessage());

            }

        }

    } finally {

        fclose($handle);

    }

    return [

        'processed' => $processed,

        'errors' => $errors,

        'time' => microtime(true) - $startTime

    ];

}

?>

```

  

**### Асинхронная обработка для длительных задач**

  

#### Фоновые задачи через exec:

```php

<?php

function runBackgroundTask($command, $outputFile = null) {

    $outputFile = $outputFile ?: '/tmp/bg_task_' . uniqid() . '.log';

    // Запуск задачи в фоне

    if (strtoupper(substr(PHP_OS, 0, 3)) === 'WIN') {

        // Windows

        $fullCommand = "start /B php $command > $outputFile 2>&1";

    } else {

        // Unix/Linux

        $fullCommand = "nohup php $command > $outputFile 2>&1 &";

    }

    exec($fullCommand, $output, $returnCode);

    return [

        'command' => $fullCommand,

        'output_file' => $outputFile,

        'return_code' => $returnCode

    ];

}

  

// Пример использования

$task = runBackgroundTask('long_running_script.php');

echo "Задача запущена, лог: " . $task['output_file'] . "\n";

?>

```

  

#### Очередь задач:

```php

<?php

class TaskQueue {

    private $queueFile;

    public function __construct($queueFile = '/tmp/task_queue.json') {

        $this->queueFile = $queueFile;

    }

    public function addTask($taskData) {

        $tasks = $this->getTasks();

        $tasks[] = [

            'id' => uniqid(),

            'data' => $taskData,

            'status' => 'pending',

            'created' => time()

        ];

        file_put_contents($this->queueFile, json_encode($tasks));

    }

    public function processQueue($maxExecutionTime = 300) {

        $startTime = time();

        $tasks = $this->getTasks();

        foreach ($tasks as &$task) {

            if ($task['status'] !== 'pending') {

                continue;

            }

            // Проверка времени

            if (time() - $startTime >= $maxExecutionTime) {

                echo "Время выполнения исчерпано, останавливаем обработку\n";

                break;

            }

            try {

                $task['status'] = 'processing';

                $this->saveTasks($tasks);

                // Выполнение задачи

                $this->executeTask($task['data']);

                $task['status'] = 'completed';

                $task['completed'] = time();

            } catch (Exception $e) {

                $task['status'] = 'failed';

                $task['error'] = $e->getMessage();

            }

            $this->saveTasks($tasks);

        }

    }

    private function executeTask($data) {

        // Логика выполнения задачи

        sleep(1); // Имитация работы

    }

    private function getTasks() {

        if (!file_exists($this->queueFile)) {

            return [];

        }

        return json_decode(file_get_contents($this->queueFile), true) ?: [];

    }

    private function saveTasks($tasks) {

        file_put_contents($this->queueFile, json_encode($tasks));

    }

}

  

// Использование

$queue = new TaskQueue();

  

// Добавление задач

$queue->addTask(['action' => 'send_email', 'email' => 'user@example.com']);

$queue->addTask(['action' => 'process_image', 'image' => 'photo.jpg']);

  

// Обработка очереди с ограничением времени

$queue->processQueue(120); // 2 минуты

?>

```

  

**### Мониторинг производительности**

  

```php

<?php

class PerformanceMonitor {

    private $startTime;

    private $checkpoints = [];

    public function __construct() {

        $this->startTime = microtime(true);

        $this->checkpoint('start');

    }

    public function checkpoint($name) {

        $current = microtime(true);

        $this->checkpoints[$name] = [

            'time' => $current,

            'elapsed' => $current - $this->startTime,

            'memory' => memory_get_usage(true),

            'peak_memory' => memory_get_peak_usage(true)

        ];

    }

    public function getReport() {

        $maxTime = ini_get('max_execution_time');

        $report = "=== Отчет о производительности ===\n";

        $report .= "Максимальное время выполнения: $maxTime сек\n\n";

        foreach ($this->checkpoints as $name => $data) {

            $report .= sprintf(

                "%s: %.4f сек (память: %s)\n",

                $name,

                $data['elapsed'],

                $this->formatBytes($data['memory'])

            );

        }

        $totalTime = end($this->checkpoints)['elapsed'];

        $remaining = $maxTime > 0 ? $maxTime - $totalTime : 'unlimited';

        $report .= "\nОбщее время: " . round($totalTime, 4) . " сек\n";

        $report .= "Оставшееся время: $remaining\n";

        return $report;

    }

    private function formatBytes($size) {

        $units = ['B', 'KB', 'MB', 'GB'];

        for ($i = 0; $size > 1024 && $i < count($units) - 1; $i++) {

            $size /= 1024;

        }

        return round($size, 2) . ' ' . $units[$i];

    }

}

  

// Пример использования

$monitor = new PerformanceMonitor();

  

// Имитация различных операций

for ($i = 0; $i < 1000; $i++) {

    // Какая-то работа

    md5(uniqid());

}

$monitor->checkpoint('hash_generation');

  

sleep(1);

$monitor->checkpoint('sleep_operation');

  

$data = range(1, 100000);

array_map('sqrt', $data);

$monitor->checkpoint('array_processing');

  

echo $monitor->getReport();

?>

```

  

**## Заключение**