**## Содержание**

  

1. [Что такое CRON и зачем он нужен](#что-такое-cron-и-зачем-он-нужен)

2. [Архитектура и компоненты CRON](#архитектура-и-компоненты-cron)

3. [Синтаксис CRON выражений](#синтаксис-cron-выражений)

4. [Управление задачами CRON](#управление-задачами-cron)

5. [Переменная окружения EDITOR](#переменная-окружения-editor)

6. [Практические примеры](#практические-примеры)

7. [Отладка и мониторинг](#отладка-и-мониторинг)

8. [Альтернативы CRON](#альтернативы-cron)

  

---

  

**## Что такое CRON и зачем он нужен**

  

**### Определение**

  

**CRON** — это демон (фоновый процесс) в Unix-подобных операционных системах, который автоматически выполняет команды или скрипты по расписанию.

  

```

┌─────────────────────────────────────┐

│         CRON Daemon (crond)         │

│   Постоянно работающий процесс      │

└─────────────────┬───────────────────┘

                  │

                  ├─ Проверка каждую минуту

                  │

    ┌─────────────┴──────────────┐

    │                            │

    ▼                            ▼

┌─────────────────┐    ┌─────────────────┐

│   System CRON   │    │   User CRON     │

│ /etc/crontab    │    │ /var/spool/cron │

│ /etc/cron.d/    │    │                 │

└─────────────────┘    └─────────────────┘

```

  

**### Зачем нужен CRON?**

  

**Основные задачи:**

  

1. **Автоматизация рутинных операций**

   - Резервное копирование данных

   - Очистка временных файлов

   - Ротация логов

   - Обновление системы

  

2. **Периодический мониторинг**

   - Проверка доступности сервисов

   - Сбор метрик

   - Проверка свободного места

   - Отправка отчетов

  

3. **Обработка данных по расписанию**

   - Генерация отчетов

   - Обработка очередей

   - Синхронизация данных

   - ETL процессы

  

4. **Техническое обслуживание**

   - Перезапуск сервисов

   - Обновление SSL сертификатов

   - Дефрагментация баз данных

   - Очистка кэша

  

**### История и происхождение**

  

```

1975 год — Создан в AT&T Unix v7

Название: от греческого "Chronos" (χρόνος) — время

Автор: Ken Thompson

```

  

**### Типы CRON систем**

  

**1. Vixie Cron** (классический, используется в большинстве дистрибутивов)

```bash

# Проверка версии

cron -V

```

  

**2. Cronie** (современная версия, в RHEL/CentOS/Fedora)

```bash

systemctl status crond

```

  

**3. Fcron** (для систем с переменным временем работы)

```bash

# Для ноутбуков, которые не работают 24/7

```

  

**4. Anacron** (для задач, которые нужно выполнить обязательно)

```bash

# Выполнит задачу при следующем запуске системы

cat /etc/anacrontab

```

  

---

  

**## Архитектура и компоненты CRON**

  

**### Структура файлов CRON**

  

```

/etc/crontab                    # Системный crontab

/etc/cron.d/                    # Системные задачи (отдельные файлы)

/etc/cron.hourly/               # Скрипты, выполняемые каждый час

/etc/cron.daily/                # Ежедневные задачи

/etc/cron.weekly/               # Еженедельные задачи

/etc/cron.monthly/              # Ежемесячные задачи

/var/spool/cron/crontabs/       # Пользовательские crontab файлы

/var/spool/cron/                # (CentOS/RHEL)

/var/log/cron                   # Логи cron (CentOS/RHEL)

/var/log/syslog                 # Логи cron (Ubuntu/Debian)

```

  

**### Системный CRON (/etc/crontab)**

  

```bash

# Просмотр системного crontab

cat /etc/crontab

```

  

**Пример содержимого:**

  

```cron

# /etc/crontab: system-wide crontab

SHELL=/bin/bash

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

MAILTO=root

HOME=/

  

# m h  dom mon dow user  command

17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly

25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )

47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )

52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

```

  

**### Пользовательский CRON**

  

**Отличия от системного:**

- Не указывается имя пользователя

- Задачи выполняются от имени владельца crontab

- Находятся в `/var/spool/cron/crontabs/username`

  

```bash

# Структура пользовательского crontab

* * * * * /path/to/command

│ │ │ │ │

│ │ │ │ └─── День недели (0-7, где 0 и 7 = воскресенье)

│ │ │ └───── Месяц (1-12)

│ │ └─────── День месяца (1-31)

│ └───────── Час (0-23)

└─────────── Минута (0-59)

```

  

**### CRON демон**

  

**Проверка статуса:**

  

```bash

# Ubuntu/Debian

sudo systemctl status cron

  

# CentOS/RHEL/Rocky

sudo systemctl status crond

  

# Запуск

sudo systemctl start cron

  

# Остановка

sudo systemctl stop cron

  

# Перезагрузка (применение изменений)

sudo systemctl restart cron

  

# Автозапуск

sudo systemctl enable cron

```

  

**Процесс CRON:**

  

```bash

# Поиск процесса

ps aux | grep cron

  

# Вывод:

# root      1234  0.0  0.1  12345  6789 ?  Ss   00:00   0:00 /usr/sbin/cron -f

  

# Проверка активности

pgrep cron

  

# Kill и перезапуск (не рекомендуется)

sudo pkill cron

sudo systemctl start cron

```

  

---

  

**## Синтаксис CRON выражений**

  

**### Основной формат**

  

```cron

┌───────────── минута (0 - 59)

│ ┌───────────── час (0 - 23)

│ │ ┌───────────── день месяца (1 - 31)

│ │ │ ┌───────────── месяц (1 - 12)

│ │ │ │ ┌───────────── день недели (0 - 7) (0 и 7 = воскресенье)

│ │ │ │ │

│ │ │ │ │

* * * * * команда для выполнения

```

  

### Специальные символы

  

| Символ | Описание | Пример |

|--------|----------|--------|

| `*` | Любое значение | `* * * * *` — каждую минуту |

| `,` | Список значений | `0 8,12,17 * * *` — в 8:00, 12:00, 17:00 |

| `-` | Диапазон | `0 9-17 * * *` — каждый час с 9 до 17 |

| `/` | Шаг (интервал) | `*/15 * * * *` — каждые 15 минут |

| `0-7` | День недели | `0=вс, 1=пн, 2=вт, 3=ср, 4=чт, 5=пт, 6=сб, 7=вс` |

  

**### Специальные строки**

  

```cron

@reboot         # При загрузке системы

@yearly         # Раз в год (0 0 1 1 *)

@annually       # То же что @yearly

@monthly        # Раз в месяц (0 0 1 * *)

@weekly         # Раз в неделю (0 0 * * 0)

@daily          # Раз в день (0 0 * * *)

@midnight       # То же что @daily

@hourly         # Каждый час (0 * * * *)

```

  

**Примеры использования:**

  

```cron

@reboot /home/user/startup.sh

@daily /usr/local/bin/backup.sh

@hourly /usr/local/bin/check-server.sh

```

  

**### Примеры CRON выражений**

  

```cron

# Каждую минуту

* * * * * /path/to/command

  

# Каждые 5 минут

*/5 * * * * /path/to/command

  

# Каждые 15 минут

*/15 * * * * /path/to/command

# или

0,15,30,45 * * * * /path/to/command

  

# Каждый час в 0 минут

0 * * * * /path/to/command

  

# Каждый день в 2:30

30 2 * * * /path/to/command

  

# Каждый понедельник в 9:00

0 9 * * 1 /path/to/command

  

# Первое число каждого месяца в 00:00

0 0 1 * * /path/to/command

  

# Каждый рабочий день в 8:00

0 8 * * 1-5 /path/to/command

  

# По выходным в 10:00

0 10 * * 0,6 /path/to/command

# или

0 10 * * 6,7 /path/to/command

  

# Каждый час в рабочее время (9-18) по будням

0 9-18 * * 1-5 /path/to/command

  

# Каждые 2 часа

0 */2 * * * /path/to/command

  

# Каждые 6 часов (00:00, 06:00, 12:00, 18:00)

0 0,6,12,18 * * * /path/to/command

  

# Каждый квартал (1 января, 1 апреля, 1 июля, 1 октября)

0 0 1 1,4,7,10 * /path/to/command

  

# Каждую полночь с понедельника по пятницу

0 0 * * 1-5 /path/to/command

  

# 1 и 15 числа каждого месяца в 14:00

0 14 1,15 * * /path/to/command

  

# Каждые 10 минут с 9:00 до 18:00 по будням

*/10 9-18 * * 1-5 /path/to/command

  

# В 23:59 каждое последнее воскресенье месяца

59 23 * * 7 [ $(date +\%d) -le 7 ] && /path/to/command

```

  

**### Сложные выражения**

  

```cron

# Только по нечетным часам

0 1,3,5,7,9,11,13,15,17,19,21,23 * * * /path/to/command

  

# Каждые 5 минут с 9:00 до 9:59 и с 17:00 до 17:59

*/5 9,17 * * * /path/to/command

  

# Понедельник и четверг в 15:30

30 15 * * 1,4 /path/to/command

  

# Первая минута первого часа первого дня первого месяца

# (каждый Новый год в 00:00)

0 0 1 1 * /path/to/newyear.sh

  

# Каждый 7-й день месяца в 3:15

15 3 7 * * /path/to/command

  

# Два раза в день: утром и вечером

0 9,21 * * * /path/to/command

```

  

**### Переменные окружения в CRON**

  

```cron

# Установка переменных

SHELL=/bin/bash

PATH=/usr/local/bin:/usr/bin:/bin

MAILTO=admin@example.com

HOME=/home/user

LANG=ru_RU.UTF-8

  

# Отключение email уведомлений

MAILTO=""

  

# Пример с переменными

0 2 * * * /usr/local/bin/backup.sh

  

# Использование своих переменных

DB_HOST=localhost

DB_USER=admin

0 3 * * * /usr/local/bin/db-backup.sh

```

  

---

  

**## Управление задачами CRON**

  

**### Отображение списка задач для текущего пользователя**

  

**Основная команда:**

  

```bash

# Просмотр своих задач

crontab -l

  

# Вывод может быть таким:

# */5 * * * * /home/user/check.sh

# 0 2 * * * /home/user/backup.sh

# @reboot /home/user/startup.sh

```

  

**Если задач нет:**

  

```bash

crontab -l

# Вывод:

# no crontab for username

```

  

**Просмотр задач другого пользователя (требует root):**

  

```bash

# От root

sudo crontab -l -u username

  

# Пример

sudo crontab -l -u www-data

```

  

**Просмотр всех пользовательских crontab:**

  

```bash

# Список файлов

sudo ls -la /var/spool/cron/crontabs/

# или в CentOS/RHEL

sudo ls -la /var/spool/cron/

  

# Просмотр содержимого

sudo cat /var/spool/cron/crontabs/username

```

  

**Просмотр системных задач:**

  

```bash

# Системный crontab

cat /etc/crontab

  

# Задачи в /etc/cron.d/

ls -la /etc/cron.d/

cat /etc/cron.d/*

  

# Скрипты в периодических директориях

ls -la /etc/cron.hourly/

ls -la /etc/cron.daily/

ls -la /etc/cron.weekly/

ls -la /etc/cron.monthly/

```

  

**### Редактирование crontab**

  

**Основная команда:**

  

```bash

# Редактирование своего crontab

crontab -e

  

# Откроется редактор (по умолчанию vi/vim или nano)

```

  

**Редактирование от другого пользователя:**

  

```bash

# От root

sudo crontab -e -u username

  

# Пример

sudo crontab -e -u www-data

```

  

**Процесс редактирования:**

  

```bash

# 1. Запуск редактора

crontab -e

  

# 2. Добавление задачи (пример)

# Каждый день в 2:00 выполнить backup

0 2 * * * /home/user/backup.sh

  

# 3. Сохранение (в vim: :wq, в nano: Ctrl+O, Enter, Ctrl+X)

  

# 4. Проверка результата

crontab -l

```

  

**Что происходит при сохранении:**

  

```bash

# CRON автоматически:

# 1. Проверяет синтаксис

# 2. Сохраняет в /var/spool/cron/crontabs/username

# 3. Перезагружает конфигурацию

# 4. Начинает выполнять задачи по расписанию

```

  

**### Удаление задач**

  

**Удаление всех задач:**

  

```bash

# ВНИМАНИЕ: удалит ВСЕ задачи без подтверждения!

crontab -r

  

# Проверка

crontab -l

# Вывод: no crontab for username

```

  

**Удаление с подтверждением (если поддерживается):**

  

```bash

crontab -r -i

# Выведет: "crontab: really delete user's crontab? (y/n)"

```

  

**Удаление конкретной задачи:**

  

```bash

# 1. Открыть редактор

crontab -e

  

# 2. Удалить строку с задачей или закомментировать

# 0 2 * * * /home/user/backup.sh  # Закомментировано

  

# 3. Сохранить

```

  

**Удаление для другого пользователя:**

  

```bash

sudo crontab -r -u username

```

  

**### Импорт/экспорт задач**

  

**Экспорт (резервная копия):**

  

```bash

# Сохранение в файл

crontab -l > my-crontab-backup.txt

  

# С датой

crontab -l > crontab-backup-$(date +%Y%m%d).txt

  

# Просмотр

cat my-crontab-backup.txt

```

  

**Импорт (восстановление):**

  

```bash

# Загрузка из файла

crontab my-crontab-backup.txt

  

# Проверка

crontab -l

```

  

**Копирование задач от одного пользователя другому:**

  

```bash

# Экспорт от user1

sudo crontab -l -u user1 > user1-crontab.txt

  

# Импорт для user2

sudo crontab -u user2 user1-crontab.txt

```

  

**### Проверка синтаксиса**

  

**Ручная проверка (нет встроенной команды в cron):**

  

```bash

# Скачать и использовать валидатор

# Онлайн: https://crontab.guru/

  

# Локальная проверка через скрипт

cat > check-cron.sh << 'EOF'

#!/bin/bash

while IFS= read -r line; do

    if [[ $line =~ ^[^#] ]]; then

        echo "Checking: $line"

    fi

done < "$1"

EOF

  

chmod +x check-cron.sh

./check-cron.sh my-crontab.txt

```

  

---

  

**## Переменная окружения EDITOR**

  

**### Что такое EDITOR?**

  

**EDITOR** — это переменная окружения, которая определяет, какой текстовый редактор будет использоваться по умолчанию для редактирования файлов в системе.

  

```bash

# Текущее значение

echo $EDITOR

  

# Если не установлена, может быть пусто или показать:

# /usr/bin/vim

# /usr/bin/nano

# /bin/vi

```

  

**### За что отвечает EDITOR?**

  

**Программы, использующие EDITOR:**

  

1. **crontab -e** — редактирование cron задач

2. **visudo** — редактирование sudoers файла

3. **git commit** — написание commit message

4. **systemctl edit** — редактирование unit файлов

5. **fc** — редактирование истории команд bash

6. **vipw** — редактирование /etc/passwd

7. **select-editor** — выбор редактора

  

**### Как использовать EDITOR?**

  

**Временная установка (на одну команду):**

  

```bash

# Использовать nano для одной команды

EDITOR=nano crontab -e

  

# Использовать vim

EDITOR=vim crontab -e

  

# Использовать emacs

EDITOR=emacs crontab -e

  

# Использовать VS Code (если установлен)

EDITOR="code --wait" crontab -e

```

  

**Установка для текущей сессии:**

  

```bash

# Установка в текущем терминале

export EDITOR=nano

  

# Проверка

echo $EDITOR

# Вывод: nano

  

# Теперь все команды будут использовать nano

crontab -e

git commit

visudo

```

  

**Постоянная установка (для пользователя):**

  

```bash

# Для bash

echo 'export EDITOR=nano' >> ~/.bashrc

source ~/.bashrc

  

# Для zsh

echo 'export EDITOR=nano' >> ~/.zshrc

source ~/.zshrc

  

# Для fish

echo 'set -gx EDITOR nano' >> ~/.config/fish/config.fish

  

# Универсальный способ (для всех shell)

echo 'export EDITOR=nano' >> ~/.profile

```

  

**Установка для всей системы (все пользователи):**

  

```bash

# Редактирование системного профиля

sudo nano /etc/environment

  

# Добавить строку

EDITOR=nano

  

# Или через /etc/profile

echo 'export EDITOR=nano' | sudo tee -a /etc/profile

```

  

**### Выбор редактора через select-editor**

  

**Ubuntu/Debian:**

  

```bash

# Интерактивный выбор редактора

select-editor

  

# Вывод:

# Select an editor.  To change later, run 'select-editor'.

#   1. /bin/nano        <---- easiest

#   2. /usr/bin/vim.basic

#   3. /usr/bin/vim.tiny

#   4. /bin/ed

#

# Choose 1-4 [1]:

  

# Выбрать номер и нажать Enter

# Настройки сохранятся в ~/.selected_editor

```

  

**Просмотр выбранного редактора:**

  

```bash

cat ~/.selected_editor

  

# Вывод:

# SELECTED_EDITOR="/usr/bin/nano"

```

  

**### Популярные редакторы**

  

**1. nano (самый простой):**

  

```bash

export EDITOR=nano

  

# Основные команды nano:

# Ctrl+O — сохранить

# Ctrl+X — выйти

# Ctrl+W — поиск

# Ctrl+K — вырезать строку

# Ctrl+U — вставить

```

  

**2. vim/vi (продвинутый):**

  

```bash

export EDITOR=vim

  

# Основные команды vim:

# i — режим вставки

# Esc — выход из режима вставки

# :w — сохранить

# :q — выйти

# :wq — сохранить и выйти

# :q! — выйти без сохранения

```

  

**3. emacs:**

  

```bash

export EDITOR=emacs

  

# Основные команды emacs:

# Ctrl+X Ctrl+S — сохранить

# Ctrl+X Ctrl+C — выйти

```

  

**4. VS Code:**

  

```bash

export EDITOR="code --wait"

  

# --wait важен! Заставляет ждать закрытия файла

```

  

**5. Sublime Text:**

  

```bash

export EDITOR="subl --wait"

```

  

**### Переменная VISUAL**

  

**Связь EDITOR и VISUAL:**

  

```bash

# VISUAL — для полноэкранных редакторов

# EDITOR — для строчных редакторов (исторически)

  

# Современная практика: установить обе

export EDITOR=nano

export VISUAL=nano

  

# Или

export VISUAL="$EDITOR"

```

  

**Приоритет:**

```

VISUAL > EDITOR > vi (по умолчанию)

```

  

**### Практические примеры**

  

**Пример 1: Смена редактора перед редактированием crontab**

  

```bash

# Вы привыкли к nano, но система использует vim

echo $EDITOR

# /usr/bin/vim

  

# Временно использовать nano

EDITOR=nano crontab -e

  

# Или установить постоянно

export EDITOR=nano

echo 'export EDITOR=nano' >> ~/.bashrc

```

  

**Пример 2: Использование graphical редактора**

  

```bash

# Установка VS Code как редактор

export EDITOR="code --wait"

  

# Теперь crontab откроется в VS Code

crontab -e

  

# Отредактировать, сохранить, закрыть вкладку

# Изменения применятся автоматически

```

  

**Пример 3: Разные редакторы для разных задач**

  

```bash

# В ~/.bashrc

export EDITOR=nano                    # По умолчанию

alias edit-cron="EDITOR=vim crontab -e"     # Для cron используем vim

alias edit-git="EDITOR=code git commit"     # Для git используем VS Code

  

# Использование

edit-cron    # Откроет crontab в vim

edit-git     # Откроет git commit в VS Code

```

  

**Пример 4: Скрипт для проверки и установки EDITOR**

  

```bash

cat > setup-editor.sh << 'EOF'

#!/bin/bash

  

echo "Текущий EDITOR: $EDITOR"

echo ""

echo "Доступные редакторы:"

echo "1. nano (простой)"

echo "2. vim (продвинутый)"

echo "3. emacs"

echo "4. code (VS Code)"

echo ""

read -p "Выберите редактор (1-4): " choice

  

case $choice in

    1) EDITOR_CMD="nano" ;;

    2) EDITOR_CMD="vim" ;;

    3) EDITOR_CMD="emacs" ;;

    4) EDITOR_CMD="code --wait" ;;

    *) echo "Неверный выбор"; exit 1 ;;

esac

  

echo "export EDITOR=$EDITOR_CMD" >> ~/.bashrc

echo "Установлен EDITOR=$EDITOR_CMD"

echo "Перезапустите терминал или выполните: source ~/.bashrc"

EOF

  

chmod +x setup-editor.sh

./setup-editor.sh

```

  

---

  

**## Практические примеры**

  

**### Резервное копирование**

  

**Ежедневный бэкап базы данных:**

  

```bash

# Создание скрипта

cat > /home/user/backup-db.sh << 'EOF'

#!/bin/bash

  

# Настройки

DB_USER="admin"

DB_PASS="password"

DB_NAME="myapp"

BACKUP_DIR="/home/user/backups"

DATE=$(date +%Y%m%d_%H%M%S)

  

# Создание директории

mkdir -p "$BACKUP_DIR"

  

# Бэкап

mysqldump -u"$DB_USER" -p"$DB_PASS" "$DB_NAME" | gzip > "$BACKUP_DIR/db_backup_$DATE.sql.gz"

  

# Удаление старых бэкапов (старше 7 дней)

find "$BACKUP_DIR" -name "db_backup_*.sql.gz" -mtime +7 -delete

  

# Логирование

echo "$(date): Backup completed" >> "$BACKUP_DIR/backup.log"

EOF

  

chmod +x /home/user/backup-db.sh

  

# Добавление в crontab (каждый день в 2:00)

crontab -e

# 0 2 * * * /home/user/backup-db.sh

```

  

**Бэкап файлов на удаленный сервер:**

  

```bash

cat > /home/user/backup-files.sh << 'EOF'

#!/bin/bash

  

SOURCE="/var/www/html"

DEST="user@backup-server:/backups/web/"

LOG="/var/log/backup.log"

  

rsync -avz --delete "$SOURCE" "$DEST" >> "$LOG" 2>&1

  

if [ $? -eq 0 ]; then

    echo "$(date): Backup successful" >> "$LOG"

else

    echo "$(date): Backup FAILED" >> "$LOG"

    # Отправка уведомления

    echo "Backup failed!" | mail -s "Backup Error" admin@example.com

fi

EOF

  

chmod +x /home/user/backup-files.sh

  

# Каждую ночь в 3:00

# 0 3 * * * /home/user/backup-files.sh

```

  

**### Мониторинг и алерты**

  

**Проверка свободного места на диске:**

  

```bash

cat > /usr/local/bin/check-disk-space.sh << 'EOF'

#!/bin/bash

  

THRESHOLD=80

EMAIL="admin@example.com"

  

df -H | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | while read output;

do

    usage=$(echo $output | awk '{print $1}' | sed 's/%//g')

    partition=$(echo $output | awk '{print $2}')

    if [ $usage -ge $THRESHOLD ]; then

        echo "Disk space alert on $partition: ${usage}% used" | \

        mail -s "Disk Space Alert: $HOSTNAME" $EMAIL

    fi

done

EOF

  

chmod +x /usr/local/bin/check-disk-space.sh

  

# Каждый час

# 0 * * * * /usr/local/bin/check-disk-space.sh

```

  

**Проверка доступности веб-сервера:**

  

```bash

cat > /usr/local/bin/check-website.sh << 'EOF'

#!/bin/bash

  

URL="https://example.com"

EMAIL="admin@example.com"

LOG="/var/log/website-check.log"

  

HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "$URL")

  

if [ "$HTTP_CODE" -ne 200 ]; then

    echo "$(date): Website DOWN! HTTP code: $HTTP_CODE" >> "$LOG"

    echo "Website $URL is DOWN! HTTP code: $HTTP_CODE" | \

    mail -s "Website Alert: DOWN" "$EMAIL"

else

    echo "$(date): Website OK (HTTP $HTTP_CODE)" >> "$LOG"

fi

EOF

  

chmod +x /usr/local/bin/check-website.sh

  

# Каждые 5 минут

# */5 * * * * /usr/local/bin/check-website.sh

```

  

**Мониторинг использования памяти:**

  

```bash

cat > /usr/local/bin/check-memory.sh << 'EOF'

#!/bin/bash

  

THRESHOLD=90

EMAIL="admin@example.com"

  

MEMORY_USAGE=$(free | grep Mem | awk '{print ($3/$2) * 100.0}' | cut -d. -f1)

  

if [ "$MEMORY_USAGE" -ge "$THRESHOLD" ]; then

    echo "Memory usage is at ${MEMORY_USAGE}%" | \

    mail -s "Memory Alert: $HOSTNAME" "$EMAIL"

fi

EOF

  

chmod +x /usr/local/bin/check-memory.sh

  

# Каждые 10 минут

# */10 * * * * /usr/local/bin/check-memory.sh

```

  

**### Очистка и обслуживание**

  

**Очистка временных файлов:**

  

```bash

cat > /usr/local/bin/cleanup-temp.sh << 'EOF'

#!/bin/bash

  

# Очистка /tmp старше 7 дней

find /tmp -type f -mtime +7 -delete

  

# Очистка логов старше 30 дней

find /var/log -name "*.log" -mtime +30 -delete

  

# Очистка кэша приложения

rm -rf /var/www/app/var/cache/*

  

# Логирование

echo "$(date): Cleanup completed" >> /var/log/cleanup.log

EOF

  

chmod +x /usr/local/bin/cleanup-temp.sh

  

# Каждый день в 4:00

# 0 4 * * * /usr/local/bin/cleanup-temp.sh

```

  

**Ротация логов (если не используется logrotate):**

  

```bash

cat > /usr/local/bin/rotate-logs.sh << 'EOF'

#!/bin/bash

  

LOG_DIR="/var/www/app/logs"

KEEP_DAYS=30

  

cd "$LOG_DIR" || exit

  

# Архивирование текущих логов

for log in *.log; do

    if [ -f "$log" ]; then

        gzip -c "$log" > "${log}.$(date +%Y%m%d).gz"

        > "$log"  # Очистка файла

    fi

done

  

# Удаление старых архивов

find . -name "*.gz" -mtime +$KEEP_DAYS -delete

EOF

  

chmod +x /usr/local/bin/rotate-logs.sh

  

# Каждое воскресенье в 5:00

# 0 5 * * 0 /usr/local/bin/rotate-logs.sh

```

  

### Автоматизация разработки

  

**Автоматическое обновление зависимостей:**

  

```bash

cat > /home/user/auto-update-deps.sh << 'EOF'

#!/bin/bash

  

PROJECT_DIR="/var/www/myapp"

LOG="/var/log/auto-update.log"

  

cd "$PROJECT_DIR" || exit

  

# Composer update

composer update >> "$LOG" 2>&1

  

# NPM update

npm update >> "$LOG" 2>&1

  

# Commit изменений

if [[ $(git status --porcelain) ]]; then

    git add composer.lock package-lock.json

    git commit -m "Auto-update: dependencies updated on $(date)"

    git push

fi

  

echo "$(date): Dependencies updated" >> "$LOG"

EOF

  

chmod +x /home/user/auto-update-deps.sh

  

# Каждое воскресенье в 3:00

# 0 3 * * 0 /home/user/auto-update-deps.sh

```

  

**Автоматическое тестирование:**

  

```bash

cat > /home/user/run-tests.sh << 'EOF'

#!/bin/bash

  

PROJECT_DIR="/var/www/myapp"

LOG="/var/log/tests.log"

EMAIL="dev@example.com"

  

cd "$PROJECT_DIR" || exit

  

# Запуск тестов

php vendor/bin/phpunit > /tmp/test-output.txt 2>&1

  

if [ $? -eq 0 ]; then

    echo "$(date): Tests PASSED" >> "$LOG"

else

    echo "$(date): Tests FAILED" >> "$LOG"

    cat /tmp/test-output.txt | mail -s "Test Failure: $HOSTNAME" "$EMAIL"

fi

EOF

  

chmod +x /home/user/run-tests.sh

  

# Каждый час в рабочее время по будням

# 0 9-18 * * 1-5 /home/user/run-tests.sh

```

  

**### Работа с очередями**

  

**Обработка очередей Laravel:**

  

```bash

# Добавление в crontab

crontab -e

  

# * * * * * cd /var/www/laravel && php artisan schedule:run >> /dev/null 2>&1

```

  

**Очистка failed jobs:**

  

```bash

cat > /home/user/cleanup-jobs.sh << 'EOF'

#!/bin/bash

  

cd /var/www/laravel || exit

  

# Очистка старых failed jobs (старше 7 дней)

php artisan queue:flush

  

# Перезапуск worker

php artisan queue:restart

EOF

  

chmod +x /home/user/cleanup-jobs.sh

  

# Каждый день в 2:00

# 0 2 * * * /home/user/cleanup-jobs.sh

```

  

**### SSL сертификаты**

  

**Обновление Let's Encrypt сертификатов:**

  

```bash

cat > /usr/local/bin/renew-ssl.sh << 'EOF'

#!/bin/bash

  

LOG="/var/log/letsencrypt-renew.log"

  

# Обновление сертификатов

certbot renew >> "$LOG" 2>&1

  

# Перезагрузка nginx

if [ $? -eq 0 ]; then

    systemctl reload nginx

    echo "$(date): SSL certificates renewed" >> "$LOG"

else

    echo "$(date): SSL renewal FAILED" >> "$LOG"

    echo "SSL renewal failed" | mail -s "SSL Alert" admin@example.com

fi

EOF

  

chmod +x /usr/local/bin/renew-ssl.sh

  

# Каждый день в 3:00 (certbot проверит, нужно ли обновление)

# 0 3 * * * /usr/local/bin/renew-ssl.sh

```

  

**### Генерация отчетов**

  

**Ежедневный отчет по логам:**

  

```bash

cat > /usr/local/bin/daily-report.sh << 'EOF'

#!/bin/bash

  

DATE=$(date +%Y-%m-%d)

REPORT="/tmp/report_$DATE.txt"

EMAIL="admin@example.com"

  

echo "Daily Server Report - $DATE" > "$REPORT"

echo "================================" >> "$REPORT"

echo "" >> "$REPORT"

  

echo "Disk Usage:" >> "$REPORT"

df -h >> "$REPORT"

echo "" >> "$REPORT"

  

echo "Memory Usage:" >> "$REPORT"

free -h >> "$REPORT"

echo "" >> "$REPORT"

  

echo "Top 10 Processes by CPU:" >> "$REPORT"

ps aux --sort=-%cpu | head -11 >> "$REPORT"

echo "" >> "$REPORT"

  

echo "Failed Login Attempts:" >> "$REPORT"

grep "Failed password" /var/log/auth.log | tail -20 >> "$REPORT"

echo "" >> "$REPORT"

  

echo "Nginx Errors:" >> "$REPORT"

grep -i error /var/log/nginx/error.log | tail -20 >> "$REPORT"

  

# Отправка отчета

cat "$REPORT" | mail -s "Daily Report: $HOSTNAME - $DATE" "$EMAIL"

  

# Удаление временного файла

rm "$REPORT"

EOF

  

chmod +x /usr/local/bin/daily-report.sh

  

# Каждый день в 23:00

# 0 23 * * * /usr/local/bin/daily-report.sh

```

  

---

  

**## Отладка и мониторинг**

  

**### Просмотр логов CRON**

  

**Ubuntu/Debian:**

  

```bash

# Просмотр логов cron

grep CRON /var/log/syslog

  

# Только сегодняшние

grep CRON /var/log/syslog | grep "$(date +%b\ %e)"

  

# Последние 50 строк

grep CRON /var/log/syslog | tail -50

  

# В реальном времени

tail -f /var/log/syslog | grep CRON

```

  

**CentOS/RHEL/Rocky:**

  

```bash

# Просмотр логов

cat /var/log/cron

  

# Последние записи

tail -50 /var/log/cron

  

# В реальном времени

tail -f /var/log/cron

  

# За конкретную дату

grep "Nov  3" /var/log/cron

```

  

**### Проверка выполнения задач**

  

**Логирование в crontab:**

  

```bash

# Вариант 1: Перенаправление в файл

* * * * * /path/to/script.sh >> /var/log/my-cron.log 2>&1

  

# Вариант 2: Отдельные файлы для stdout и stderr

* * * * * /path/to/script.sh >> /var/log/my-cron.log 2>> /var/log/my-cron-errors.log

  

# Вариант 3: С меткой времени

* * * * * echo "$(date): Starting script" >> /var/log/my-cron.log && /path/to/script.sh >> /var/log/my-cron.log 2>&1

```

  

**Пример скрипта с логированием:**

  

```bash

cat > /home/user/logged-script.sh << 'EOF'

#!/bin/bash

  

LOG="/var/log/my-script.log"

  

log() {

    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> "$LOG"

}

  

log "Script started"

  

# Ваш код

log "Processing data..."

# ...

  

log "Script completed"

EOF

  

chmod +x /home/user/logged-script.sh

```

  

**### Отладка не работающих задач**

  

**Проверочный список:**

  

```bash

# 1. Проверить, работает ли CRON демон

systemctl status cron

  

# 2. Проверить синтаксис crontab

crontab -l

  

# 3. Проверить права на скрипт

ls -l /path/to/script.sh

# Должен быть исполняемым: -rwxr-xr-x

  

# 4. Проверить путь к интерпретатору

head -1 /path/to/script.sh

# Должно быть: #!/bin/bash

  

# 5. Проверить переменные окружения

# Создать тестовый скрипт

cat > /tmp/test-env.sh << 'EOF'

#!/bin/bash

env > /tmp/cron-env.txt

EOF

  

chmod +x /tmp/test-env.sh

  

# Добавить в crontab

# * * * * * /tmp/test-env.sh

  

# Через минуту проверить

cat /tmp/cron-env.txt

  

# 6. Тестовая задача

# * * * * * echo "Test at $(date)" >> /tmp/cron-test.log

```

  

**Типичные проблемы:**

  

```bash

# Проблема 1: Относительные пути

# НЕПРАВИЛЬНО:

* * * * * ./script.sh

  

# ПРАВИЛЬНО:

* * * * * /home/user/script.sh

  

# Проблема 2: PATH не установлен

# НЕПРАВИЛЬНО:

* * * * * php script.php

  

# ПРАВИЛЬНО:

* * * * * /usr/bin/php /home/user/script.php

  

# Или установить PATH в crontab:

PATH=/usr/local/bin:/usr/bin:/bin

* * * * * php /home/user/script.php

  

# Проблема 3: Отсутствие прав

chmod +x /path/to/script.sh

  

# Проблема 4: Неправильный shebang

# В начале скрипта должно быть:

#!/bin/bash

```

  

**### Email уведомления**

  

**Настройка MAILTO:**

  

```bash

crontab -e

  

# Отправлять все на email

MAILTO=admin@example.com

  

# Отключить email

MAILTO=""

  

# Отправлять только ошибки

* * * * * /path/to/script.sh > /dev/null

# stderr все равно отправится на email

  

# Отключить все уведомления

* * * * * /path/to/script.sh > /dev/null 2>&1

```

  

**Кастомные уведомления:**

  

```bash

cat > /home/user/notify-script.sh << 'EOF'

#!/bin/bash

  

OUTPUT=$(/path/to/command 2>&1)

STATUS=$?

  

if [ $STATUS -ne 0 ]; then

    echo "$OUTPUT" | mail -s "Script Failed on $HOSTNAME" admin@example.com

fi

EOF

  

chmod +x /home/user/notify-script.sh

```

  

**### Мониторинг CRON задач**

  

**Создание мониторинга:**

  

```bash

cat > /usr/local/bin/cron-monitor.sh << 'EOF'

#!/bin/bash

  

echo "=== CRON Monitor ==="

echo ""

  

# Статус демона

echo "CRON Daemon Status:"

systemctl status cron --no-pager | head -5

echo ""

  

# Количество задач

echo "Number of User Crontabs:"

ls -1 /var/spool/cron/crontabs/ 2>/dev/null | wc -l

echo ""

  

# Последние выполнения

echo "Last 10 CRON Executions:"

grep CRON /var/log/syslog 2>/dev/null | tail -10

echo ""

  

# Активные задачи сейчас

echo "Currently Running CRON Jobs:"

ps aux | grep -E "CRON|cron" | grep -v grep

EOF

  

chmod +x /usr/local/bin/cron-monitor.sh

  

# Запуск

/usr/local/bin/cron-monitor.sh

```

  

---

  

**## Альтернативы CRON**

  

**### 1. Systemd Timers**

  

**Современная альтернатива в systemd-based системах:**

  

```bash

# Создание service файла

sudo nano /etc/systemd/system/my-backup.service

```

  

```ini

[Unit]

Description=My Backup Service

After=network.target

  

[Service]

Type=oneshot

ExecStart=/home/user/backup.sh

User=user

```

  

```bash

# Создание timer файла

sudo nano /etc/systemd/system/my-backup.timer

```

  

```ini

[Unit]

Description=Run backup daily

Requires=my-backup.service

  

[Timer]

OnCalendar=daily

Persistent=true

  

[Install]

WantedBy=timers.target

```

  

```bash

# Активация

sudo systemctl daemon-reload

sudo systemctl enable my-backup.timer

sudo systemctl start my-backup.timer

  

# Проверка статуса

systemctl list-timers --all

systemctl status my-backup.timer

```

  

**Преимущества systemd timers:**

- Лучшее логирование (journalctl)

- Зависимости между задачами

- Точность до микросекунд

- Автоматический перезапуск

  

**### 2. Anacron**

  

**Для систем, работающих не 24/7:**

  

```bash

# Установка

sudo apt install anacron

  

# Конфигурация /etc/anacrontab

cat /etc/anacrontab

```

  

```bash

# period  delay  job-identifier  command

1         5      cron.daily      run-parts /etc/cron.daily

7         10     cron.weekly     run-parts /etc/cron.weekly

30        15     cron.monthly    run-parts /etc/cron.monthly

```

  

**Преимущества:**

- Выполнит задачу после включения, если пропущена

- Идеально для ноутбуков/десктопов

  

**### 3. At (однократное выполнение)**

  

```bash

# Установка

sudo apt install at

  

# Запуск через 5 минут

echo "/path/to/script.sh" | at now + 5 minutes

  

# Запуск в конкретное время

at 14:30

at> /path/to/script.sh

at> <Ctrl+D>

  

# Список задач

atq

  

# Удаление задачи

atrm <job_number>

```

  

**### 4. Fcron**

  

**Продвинутая версия cron:**

  

```bash

# Установка

sudo apt install fcron

  

# Возможности:

# - Выполнение пропущенных задач

# - Серийное выполнение

# - Временные зоны

```

  

---

  

**## Заключение**

  

**### Лучшие практики**

  

✅ **DO:**

- Всегда использовать абсолютные пути

- Логировать выполнение задач

- Устанавливать MAILTO или направлять вывод в /dev/null

- Тестировать скрипты вручную перед добавлением в cron

- Делать резервные копии crontab

- Использовать комментарии в crontab

- Проверять права доступа к скриптам

  

❌ **DON'T:**

- Запускать задачи слишком часто без необходимости

- Использовать относительные пути

- Игнорировать ошибки в логах

- Запускать критичные задачи без мониторинга

- Хранить пароли в открытом виде

  

**### Полезные ресурсы**

  

**Онлайн инструменты:**

- https://crontab.guru/ — визуальный редактор cron выражений

- https://crontab-generator.org/ — генератор crontab

- https://cronitor.io/ — мониторинг cron задач

  

**Документация:**

```bash

man crontab

man 5 crontab

man cron

man anacron

```

  

**### Быстрая шпаргалка**