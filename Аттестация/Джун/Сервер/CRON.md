# CRON

  

## Содержание

  

- [[#Что такое CRON и зачем он нужен]]

- [[#Архитектура и компоненты CRON]]

- [[#Синтаксис CRON выражений]]

- [[#Управление задачами CRON]]

- [[#Переменная окружения EDITOR]]

- [[#Практические примеры]]

- [[#Отладка и мониторинг]]

- [[#Альтернативы CRON]]

- [[#Быстрая шпаргалка]]

  

---

  

## Что такое CRON и зачем он нужен

  

### Определение

  

**CRON** — это демон (фоновый процесс) в Unix-подобных операционных системах, который автоматически выполняет команды или скрипты по расписанию.

  

```

┌─────────────────────────────────────┐

│ CRON Daemon (crond) │

│ Постоянно работающий процесс │

└─────────────────┬───────────────────┘

│

├─ Проверка каждую минуту

│

┌─────────────┴──────────────┐

│ │

▼ ▼

┌─────────────────┐ ┌─────────────────┐

│ System CRON │ │ User CRON │

│ /etc/crontab │ │ /var/spool/cron │

│ /etc/cron.d/ │ │ │

└─────────────────┘ └─────────────────┘

```

  

### Зачем нужен CRON?

  

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

  

### История и происхождение

  

```

1975 год — Создан в AT&T Unix v7

Название: от греческого "Chronos" (χρόνος) — время

Автор: Ken Thompson

```

  

### Типы CRON систем

  

**1. Vixie Cron** (классический, используется в большинстве дистрибутивов)

  

```bash

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

cat /etc/anacrontab

```

  

---

  

## Архитектура и компоненты CRON

  

### Структура файлов CRON

  

```

/etc/crontab # Системный crontab

/etc/cron.d/ # Системные задачи (отдельные файлы)

/etc/cron.hourly/ # Скрипты, выполняемые каждый час

/etc/cron.daily/ # Ежедневные задачи

/etc/cron.weekly/ # Еженедельные задачи

/etc/cron.monthly/ # Ежемесячные задачи

/var/spool/cron/crontabs/ # Пользовательские crontab файлы

/var/spool/cron/ # (CentOS/RHEL)

/var/log/cron # Логи cron (CentOS/RHEL)

/var/log/syslog # Логи cron (Ubuntu/Debian)

```

  

### Системный CRON (`/etc/crontab`)

  

```bash

cat /etc/crontab

```

  

Пример содержимого:

  

```cron

# /etc/crontab: system-wide crontab

SHELL=/bin/bash

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

MAILTO=root

HOME=/

  

# m h dom mon dow user command

17 * * * * root cd / && run-parts --report /etc/cron.hourly

25 6 * * * root test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )

47 6 * * 7 root test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )

52 6 1 * * root test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

```

  

### Пользовательский CRON

  

**Отличия от системного:**

- Не указывается имя пользователя

- Задачи выполняются от имени владельца crontab

- Находятся в `/var/spool/cron/crontabs/username`

  

```

* * * * * /path/to/command

│ │ │ │ │

│ │ │ │ └─── День недели (0-7, где 0 и 7 = воскресенье)

│ │ │ └───── Месяц (1-12)

│ │ └─────── День месяца (1-31)

│ └───────── Час (0-23)

└─────────── Минута (0-59)

```

  

### CRON демон

  

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

  

# Проверка активности

pgrep cron

  

# Kill и перезапуск (не рекомендуется)

sudo pkill cron

sudo systemctl start cron

```

  

---

  

## Синтаксис CRON выражений

  

### Основной формат

  

```

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

  

### Специальные строки

  

```cron

@reboot # При загрузке системы

@yearly # Раз в год (0 0 1 1 *)

@annually # То же что @yearly

@monthly # Раз в месяц (0 0 1 * *)

@weekly # Раз в неделю (0 0 * * 0)

@daily # Раз в день (0 0 * * *)

@midnight # То же что @daily

@hourly # Каждый час (0 * * * *)

```

  

Примеры:

  

```cron

@reboot /home/user/startup.sh

@daily /usr/local/bin/backup.sh

@hourly /usr/local/bin/check-server.sh

```

  

### Примеры CRON выражений

  

```cron

# Каждую минуту

* * * * * /path/to/command

  

# Каждые 5 минут

*/5 * * * * /path/to/command

  

# Каждые 15 минут

*/15 * * * * /path/to/command

  

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

  

### Сложные выражения

  

```cron

# Только по нечетным часам

0 1,3,5,7,9,11,13,15,17,19,21,23 * * * /path/to/command

  

# Каждые 5 минут с 9:00 до 9:59 и с 17:00 до 17:59

*/5 9,17 * * * /path/to/command

  

# Понедельник и четверг в 15:30

30 15 * * 1,4 /path/to/command

  

# Каждый Новый год в 00:00

0 0 1 1 * /path/to/newyear.sh

  

# Каждый 7-й день месяца в 3:15

15 3 7 * * /path/to/command

  

# Два раза в день: утром и вечером

0 9,21 * * * /path/to/command

```

  

### Переменные окружения в CRON

  

```cron

SHELL=/bin/bash

PATH=/usr/local/bin:/usr/bin:/bin

MAILTO=admin@example.com

HOME=/home/user

LANG=ru_RU.UTF-8

  

# Отключение email уведомлений

MAILTO=""

  

DB_HOST=localhost

DB_USER=admin

0 3 * * * /usr/local/bin/db-backup.sh

```

  

---

  

## Управление задачами CRON

  

### Отображение списка задач

  

```bash

# Просмотр своих задач

crontab -l

  

# Просмотр задач другого пользователя (требует root)

sudo crontab -l -u username

  

# Просмотр всех пользовательских crontab

sudo ls -la /var/spool/cron/crontabs/

sudo cat /var/spool/cron/crontabs/username

```

  

**Просмотр системных задач:**

  

```bash

cat /etc/crontab

ls -la /etc/cron.d/

cat /etc/cron.d/*

ls -la /etc/cron.hourly/

ls -la /etc/cron.daily/

ls -la /etc/cron.weekly/

ls -la /etc/cron.monthly/

```

  

### Редактирование crontab

  

```bash

# Редактирование своего crontab

crontab -e

  

# Редактирование от другого пользователя

sudo crontab -e -u username

```

  

При сохранении CRON автоматически:

1. Проверяет синтаксис

2. Сохраняет в `/var/spool/cron/crontabs/username`

3. Перезагружает конфигурацию

4. Начинает выполнять задачи по расписанию

  

### Удаление задач

  

```bash

# Удаление ВСЕХ задач без подтверждения

crontab -r

  

# Удаление с подтверждением

crontab -r -i

  

# Удаление конкретной задачи — через crontab -e, удалить строку

  

# Удаление для другого пользователя

sudo crontab -r -u username

```

  

### Импорт/экспорт задач

  

```bash

# Экспорт (резервная копия)

crontab -l > my-crontab-backup.txt

crontab -l > crontab-backup-$(date +%Y%m%d).txt

  

# Импорт (восстановление)

crontab my-crontab-backup.txt

  

# Копирование задач от одного пользователя другому

sudo crontab -l -u user1 > user1-crontab.txt

sudo crontab -u user2 user1-crontab.txt

```

  

---

  

## Переменная окружения EDITOR

  

### Что такое EDITOR?

  

**EDITOR** — переменная окружения, определяющая текстовый редактор по умолчанию.

  

```bash

echo $EDITOR

# /usr/bin/vim | /usr/bin/nano | /bin/vi

```

  

Программы, использующие EDITOR: `crontab -e`, `visudo`, `git commit`, `systemctl edit`, `fc`, `vipw`

  

### Установка EDITOR

  

**Временная (на одну команду):**

  

```bash

EDITOR=nano crontab -e

EDITOR="code --wait" crontab -e

```

  

**Для текущей сессии:**

  

```bash

export EDITOR=nano

```

  

**Постоянная (для пользователя):**

  

```bash

# bash

echo 'export EDITOR=nano' >> ~/.bashrc

source ~/.bashrc

  

# zsh

echo 'export EDITOR=nano' >> ~/.zshrc

source ~/.zshrc

  

# Универсальный способ

echo 'export EDITOR=nano' >> ~/.profile

```

  

**Для всей системы:**

  

```bash

sudo nano /etc/environment

# Добавить: EDITOR=nano

```

  

### select-editor (Ubuntu/Debian)

  

```bash

select-editor

# Интерактивный выбор из списка

  

cat ~/.selected_editor

# SELECTED_EDITOR="/usr/bin/nano"

```

  

### Популярные редакторы

  

| Редактор | Установка | Ключевые комбинации |

|----------|-----------|-------------------|

| **nano** | `export EDITOR=nano` | `Ctrl+O` сохранить, `Ctrl+X` выход |

| **vim** | `export EDITOR=vim` | `i` вставка, `Esc` выход, `:wq` сохранить и выйти |

| **emacs** | `export EDITOR=emacs` | `Ctrl+X Ctrl+S` сохранить, `Ctrl+X Ctrl+C` выход |

| **VS Code** | `export EDITOR="code --wait"` | Нужен флаг `--wait` |

| **Sublime** | `export EDITOR="subl --wait"` | Нужен флаг `--wait` |

  

### Переменная VISUAL

  

Приоритет: `VISUAL > EDITOR > vi`

  

```bash

export EDITOR=nano

export VISUAL="$EDITOR"

```

  

---

  

## Практические примеры

  

### Резервное копирование

  

**Ежедневный бэкап базы данных:**

  

```bash

cat > /home/user/backup-db.sh << 'EOF'

#!/bin/bash

DB_USER="admin"

DB_PASS="password"

DB_NAME="myapp"

BACKUP_DIR="/home/user/backups"

DATE=$(date +%Y%m%d_%H%M%S)

  

mkdir -p "$BACKUP_DIR"

mysqldump -u"$DB_USER" -p"$DB_PASS" "$DB_NAME" | gzip > "$BACKUP_DIR/db_backup_$DATE.sql.gz"

find "$BACKUP_DIR" -name "db_backup_*.sql.gz" -mtime +7 -delete

echo "$(date): Backup completed" >> "$BACKUP_DIR/backup.log"

EOF

  

chmod +x /home/user/backup-db.sh

# crontab: 0 2 * * * /home/user/backup-db.sh

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

echo "Backup failed!" | mail -s "Backup Error" admin@example.com

fi

EOF

  

chmod +x /home/user/backup-files.sh

# crontab: 0 3 * * * /home/user/backup-files.sh

```

  

### Мониторинг и алерты

  

**Проверка свободного места:**

  

```bash

cat > /usr/local/bin/check-disk-space.sh << 'EOF'

#!/bin/bash

THRESHOLD=80

EMAIL="admin@example.com"

  

df -H | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | while read output; do

usage=$(echo $output | awk '{print $1}' | sed 's/%//g')

partition=$(echo $output | awk '{print $2}')

if [ $usage -ge $THRESHOLD ]; then

echo "Disk space alert on $partition: ${usage}% used" | \

mail -s "Disk Space Alert: $HOSTNAME" $EMAIL

fi

done

EOF

  

chmod +x /usr/local/bin/check-disk-space.sh

# crontab: 0 * * * * /usr/local/bin/check-disk-space.sh

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

echo "Website $URL is DOWN! HTTP code: $HTTP_CODE" | mail -s "Website Alert" "$EMAIL"

else

echo "$(date): Website OK (HTTP $HTTP_CODE)" >> "$LOG"

fi

EOF

  

chmod +x /usr/local/bin/check-website.sh

# crontab: */5 * * * * /usr/local/bin/check-website.sh

```

  

### Очистка и обслуживание

  

```bash

cat > /usr/local/bin/cleanup-temp.sh << 'EOF'

#!/bin/bash

find /tmp -type f -mtime +7 -delete

find /var/log -name "*.log" -mtime +30 -delete

rm -rf /var/www/app/var/cache/*

echo "$(date): Cleanup completed" >> /var/log/cleanup.log

EOF

  

chmod +x /usr/local/bin/cleanup-temp.sh

# crontab: 0 4 * * * /usr/local/bin/cleanup-temp.sh

```

  

### SSL сертификаты

  

```bash

cat > /usr/local/bin/renew-ssl.sh << 'EOF'

#!/bin/bash

LOG="/var/log/letsencrypt-renew.log"

  

certbot renew >> "$LOG" 2>&1

if [ $? -eq 0 ]; then

systemctl reload nginx

echo "$(date): SSL certificates renewed" >> "$LOG"

else

echo "$(date): SSL renewal FAILED" >> "$LOG"

echo "SSL renewal failed" | mail -s "SSL Alert" admin@example.com

fi

EOF

  

chmod +x /usr/local/bin/renew-ssl.sh

# crontab: 0 3 * * * /usr/local/bin/renew-ssl.sh

```

  

### Laravel Scheduler

  

```cron

* * * * * cd /var/www/laravel && php artisan schedule:run >> /dev/null 2>&1

```

  

---

  

## Отладка и мониторинг

  

### Просмотр логов CRON

  

```bash

# Ubuntu/Debian

grep CRON /var/log/syslog

grep CRON /var/log/syslog | tail -50

tail -f /var/log/syslog | grep CRON

  

# CentOS/RHEL/Rocky

cat /var/log/cron

tail -50 /var/log/cron

tail -f /var/log/cron

```

  

### Логирование в crontab

  

```cron

# Перенаправление в файл

* * * * * /path/to/script.sh >> /var/log/my-cron.log 2>&1

  

# Отдельные файлы для stdout и stderr

* * * * * /path/to/script.sh >> /var/log/my-cron.log 2>> /var/log/my-cron-errors.log

  

# С меткой времени

* * * * * echo "$(date): Starting" >> /var/log/my-cron.log && /path/to/script.sh >> /var/log/my-cron.log 2>&1

```

  

### Чеклист: задача не работает

  

```bash

# 1. Работает ли CRON демон?

systemctl status cron

  

# 2. Правильный ли синтаксис?

crontab -l

  

# 3. Есть ли права на исполнение?

ls -l /path/to/script.sh

  

# 4. Есть ли shebang?

head -1 /path/to/script.sh # Должно быть #!/bin/bash

  

# 5. Используются ли абсолютные пути?

# НЕПРАВИЛЬНО: * * * * * ./script.sh

# ПРАВИЛЬНО: * * * * * /home/user/script.sh

  

# 6. Доступны ли команды в PATH?

# НЕПРАВИЛЬНО: * * * * * php script.php

# ПРАВИЛЬНО: * * * * * /usr/bin/php /home/user/script.php

# Или добавьте PATH в crontab

  

# 7. Тестовая задача для проверки

# * * * * * echo "Test at $(date)" >> /tmp/cron-test.log

```

  

### Email уведомления

  

```cron

MAILTO=admin@example.com # Отправлять все на email

MAILTO="" # Отключить email

  

* * * * * /path/to/script.sh > /dev/null # Только stderr на email

* * * * * /path/to/script.sh > /dev/null 2>&1 # Полная тишина

```

  

---

  

## Альтернативы CRON

  

### Systemd Timers

  

```ini

# /etc/systemd/system/my-backup.service

[Unit]

Description=My Backup Service

After=network.target

  

[Service]

Type=oneshot

ExecStart=/home/user/backup.sh

User=user

```

  

```ini

# /etc/systemd/system/my-backup.timer

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

sudo systemctl daemon-reload

sudo systemctl enable my-backup.timer

sudo systemctl start my-backup.timer

systemctl list-timers --all

```

  

> [!tip] Преимущества systemd timers

> - Лучшее логирование (`journalctl`)

> - Зависимости между задачами

> - Точность до микросекунд

> - Автоматический перезапуск при сбоях

  

### Anacron (для систем не 24/7)

  

```bash

sudo apt install anacron

cat /etc/anacrontab

```

  

```bash

# period delay job-identifier command

1 5 cron.daily run-parts /etc/cron.daily

7 10 cron.weekly run-parts /etc/cron.weekly

30 15 cron.monthly run-parts /etc/cron.monthly

```

  

### At (однократное выполнение)

  

```bash

sudo apt install at

  

echo "/path/to/script.sh" | at now + 5 minutes

at 14:30

at> /path/to/script.sh

at> <Ctrl+D>

  

atq # Список задач

atrm 3 # Удаление задачи №3

```

  

---

  

## Быстрая шпаргалка

  

### Команды crontab

  

| Флаг | Описание |

|------|----------|

| `crontab -l` | Показать задачи |

| `crontab -e` | Редактировать задачи |

| `crontab -r` | Удалить все задачи |

| `crontab -r -i` | Удалить с подтверждением |

| `crontab -u user -l` | Задачи пользователя (root) |

| `crontab file.txt` | Импорт из файла |

  

### Формат выражения

  

```

┌─ минута (0-59)

│ ┌─ час (0-23)

│ │ ┌─ день (1-31)

│ │ │ ┌─ месяц (1-12)

│ │ │ │ ┌─ день недели (0-7, 0=вс)

* * * * * команда

```

  

### Частые интервалы

  

| Выражение | Значение |

|-----------|----------|

| `*/5 * * * *` | Каждые 5 минут |

| `*/15 * * * *` | Каждые 15 минут |

| `0 * * * *` | Каждый час |

| `0 */2 * * *` | Каждые 2 часа |

| `0 0 * * *` | Каждый день в полночь |

| `0 2 * * *` | Каждый день в 2:00 |

| `0 0 * * 0` | Каждое воскресенье |

| `0 0 * * 1-5` | Каждый будний день |

| `0 0 1 * *` | Первое число месяца |

| `0 0 1 1 *` | Раз в год (1 января) |

  

### Специальные строки

  

| Строка | Эквивалент | Значение |

|--------|-----------|----------|

| `@reboot` | — | При загрузке |

| `@hourly` | `0 * * * *` | Каждый час |

| `@daily` | `0 0 * * *` | Каждый день |

| `@weekly` | `0 0 * * 0` | Каждую неделю |

| `@monthly` | `0 0 1 * *` | Каждый месяц |

| `@yearly` | `0 0 1 1 *` | Каждый год |

  

### Управление демонами

  

| Действие | Debian/Ubuntu | RHEL/CentOS |

|----------|---------------|-------------|

| Статус | `systemctl status cron` | `systemctl status crond` |

| Запуск | `systemctl start cron` | `systemctl start crond` |

| Остановка | `systemctl stop cron` | `systemctl stop crond` |

| Перезапуск | `systemctl restart cron` | `systemctl restart crond` |

| Автозапуск | `systemctl enable cron` | `systemctl enable crond` |

  

### Типичные ошибки

  

| Проблема | Решение |

|----------|---------|

| Относительные пути | Использовать абсолютные пути |

| Команда не найдена | Указать полный путь или `PATH=` в crontab |

| Нет прав на исполнение | `chmod +x script.sh` |

| Нет shebang | Добавить `#!/bin/bash` в начало скрипта |

| Нет вывода в лог | Добавить `>> /var/log/cron.log 2>&1` |

| Спецсимволы `%` | Экранировать: `\%` |

  

> [!warning] Best Practices

> - Всегда используйте абсолютные пути

> - Логируйте выполнение задач

> - Устанавливайте `MAILTO` или направляйте вывод в `/dev/null`

> - Тестируйте скрипты вручную перед добавлением в cron

> - Делайте резервные копии crontab (`crontab -l > backup.txt`)

> - Не храните пароли в открытом виде

  

> [!info] Полезные ссылки

> - [crontab.guru](https://crontab.guru/) — визуальный редактор cron выражений

> - [crontab-generator.org](https://crontab-generator.org/) — генератор crontab

> - `man 5 crontab` — документация по формату crontab

> - `man cron` — документация по демону

  

### Связанные темы

  

- [[Linux Управление процессами]]

- [[Linux Файловая система]]