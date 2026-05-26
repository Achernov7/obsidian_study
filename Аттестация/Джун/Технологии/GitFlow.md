---

aliases:

- Git Flow

- git-flow

- модель ветвления Git

tags:

- git

- gitflow

- branching

- devops

- ci-cd

created: 2026-05-26

---

  

# GitFlow

  

> [!info] Ссылки по теме

> - [[GitHub Flow]] — более простая альтернатива

> - [[Trunk-Based Development]] — разработка по магистрали

> - [[GitLab Flow]] — модель с environment-ветками

> - [[Семантическое версионирование]] — versioning для релизов

> - [Оригинальная статья Vincent Driessen (2010)](https://nvie.com/posts/a-successful-git-branching-model/)

  

## Что такое GitFlow

  

**GitFlow** — это модель ветвления (branching model) для Git, предложенная Vincent Driessen в 2010 году. Это набор правил и соглашений о том, как организовать работу с ветками в Git для эффективной командной разработки и управления релизами.

  

GitFlow определяет строгую структуру веток и чёткие правила их использования, что делает процесс разработки предсказуемым и управляемым, особенно для проектов с регулярными релизами.

  

## Какую проблему решает GitFlow

  

### Проблемы без структурированного подхода

  

**1. Хаос в ветках**

  

```

master

├─ feature-login

├─ bugfix-payment

├─ johns-branch

├─ test-something

├─ fix-urgent

└─ new-feature-v2-final-really-final

```

  

Без системы сложно понять:

- Что находится в каждой ветке

- Какие изменения готовы к релизу

- Где исправлять критические баги

- Как откатиться к предыдущей версии

  

**2. Конфликты при релизах**

  

```

Developer 1: добавил новую фичу → merge в master

Developer 2: исправил баг → merge в master

Developer 3: начал эксперимент → merge в master

→ Продакшн сломан, непонятно что откатывать

```

  

**3. Невозможность параллельной разработки**

  

- Нельзя одновременно работать над новыми фичами и исправлять баги в продакшне

- Блокировка релизов из-за незавершённых фич

- Сложность тестирования отдельных фич

  

**4. Проблемы с hotfix**

  

Критический баг в продакшне, а в master уже новые недотестированные фичи:

  

```

master: [v1.0] → [feature A] → [feature B] → [feature C (broken)]

↑

Нужен hotfix, но как?

```

  

**5. Отсутствие истории релизов**

  

- Непонятно, что было в каком релизе

- Сложно откатиться к стабильной версии

- Нет чёткого понимания, что идёт в следующий релиз

  

### Как GitFlow решает эти проблемы

  

> [!tip] Ключевые преимущества

> 1. **Чёткая структура веток** — каждая ветка имеет своё назначение и жизненный цикл

> 2. **Изоляция разработки** — новые фичи не влияют на стабильную версию; можно тестировать фичи независимо; hotfix'ы не смешиваются с новой разработкой

> 3. **Параллельная работа** — разработка новых фич, подготовка релиза и исправление багов в продакшне происходят одновременно без конфликтов

> 4. **Управление релизами** — понятно что идёт в следующий релиз; можно заморозить фичи для стабилизации; история версий в виде тегов

> 5. **Быстрая реакция на проблемы** — hotfix создаётся от стабильной продакшн версии; применяется и в продакшн, и в разработку; не затрагивает незавершённые фичи

  

---

  

## Структура веток в GitFlow

  

### Основные (постоянные) ветки

  

#### `master` (или `main`)

  

> [!note] Назначение

> - Содержит только **production-ready** код

> - Всегда стабильная и рабочая версия

> - Каждый коммит — это новая версия в продакшне

> - Автоматически деплоится на production

  

> [!warning] Правила

> - Никогда не коммитить напрямую

> - Только merge из `release` или `hotfix` веток

> - Каждый merge помечается тегом версии (`v1.0.0`, `v1.1.0`)

  

**Теги:**

  

```gitgraph

master

v1.0.0 → [commit] → v1.0.1 → [commit] → v1.1.0 → [commit] → v2.0.0

```

  

#### `develop`

  

> [!note] Назначение

> - Ветка интеграции всех завершённых фич

> - Содержит код для следующего релиза

> - Автоматически деплоится на dev/staging окружение

  

> [!warning] Правила

> - Принимает merge из `feature` веток

> - Должна быть стабильной (проходить тесты)

> - Отсюда создаются `release` ветки

> - Базовая ветка для новых `feature` веток

  

### Вспомогательные (временные) ветки

  

#### `feature/*` — ветки фич

  

> [!note] Назначение

> - Разработка новой функциональности

> - Изолированная работа над задачей

> - Может жить долго (недели/месяцы)

  

**Именование:**

  

```

feature/user-authentication

feature/payment-integration

feature/JIRA-123-add-comments

feature/redesign-dashboard

```

  

**Жизненный цикл:**

  

```bash

# Создание от develop

git checkout develop

git checkout -b feature/new-feature

  

# Разработка

git add .

git commit -m "Add new feature"

  

# Обновление от develop (если долгая разработка)

git checkout develop

git pull

git checkout feature/new-feature

git merge develop

  

# Завершение — merge в develop

git checkout develop

git merge --no-ff feature/new-feature

git branch -d feature/new-feature

git push origin develop

```

  

> [!abstract] Правила ветки `feature/*`

> | Поле | Значение |

> |------|----------|

> | Создаётся от | `develop` |

> | Merge в | `develop` |

> | Именование | `feature/*` |

> | Удаляется | после merge |

  

#### `release/*` — ветки релизов

  

> [!note] Назначение

> - Подготовка к новому релизу

> - Финальное тестирование

> - Исправление мелких багов

> - Обновление версии, changelog, документации

> - Замораживание функциональности

  

**Именование:**

  

```

release/1.2.0

release/v2.0.0

release/2024-Q1

```

  

**Жизненный цикл:**

  

```bash

# Создание от develop

git checkout develop

git checkout -b release/1.2.0

  

# Обновление версии

echo "1.2.0" > VERSION

git commit -a -m "Bump version to 1.2.0"

  

# Исправление багов, найденных при тестировании

git commit -a -m "Fix: Minor bugs in release"

  

# Завершение релиза

# 1. Merge в master

git checkout master

git merge --no-ff release/1.2.0

git tag -a v1.2.0 -m "Release version 1.2.0"

  

# 2. Merge обратно в develop

git checkout develop

git merge --no-ff release/1.2.0

  

# 3. Удаление ветки релиза

git branch -d release/1.2.0

  

# Push

git push origin master develop --tags

```

  

> [!abstract] Правила ветки `release/*`

> | Поле | Значение |

> |------|----------|

> | Создаётся от | `develop` |

> | Merge в | `master` **И** `develop` |

> | Именование | `release/*` |

> | Удаляется | после merge |

> | Создаётся когда | develop готов к релизу |

> | Разрешено | только bugfix, никаких новых фич |

  

> [!danger] Важно

> В `release` ветке **запрещено** добавлять новые фичи — только исправление багов и подготовка к релизу.

  

#### `hotfix/*` — ветки срочных исправлений

  

> [!note] Назначение

> - Критические баги в production

> - Срочные исправления

> - Обход обычного цикла разработки

> - Минимальные изменения

  

**Именование:**

  

```

hotfix/1.2.1

hotfix/critical-payment-bug

hotfix/security-patch

```

  

**Жизненный цикл:**

  

```bash

# Создание от master (от текущей продакшн версии!)

git checkout master

git checkout -b hotfix/1.2.1

  

# Исправление

git commit -a -m "Fix critical bug in payment system"

  

# Обновление версии

echo "1.2.1" > VERSION

git commit -a -m "Bump version to 1.2.1"

  

# Завершение hotfix

# 1. Merge в master

git checkout master

git merge --no-ff hotfix/1.2.1

git tag -a v1.2.1 -m "Hotfix version 1.2.1"

  

# 2. Merge в develop (или в текущую release ветку, если она существует)

git checkout develop

git merge --no-ff hotfix/1.2.1

  

# 3. Удаление ветки

git branch -d hotfix/1.2.1

  

# Push

git push origin master develop --tags

```

  

> [!abstract] Правила ветки `hotfix/*`

> | Поле | Значение |

> |------|----------|

> | Создаётся от | `master` |

> | Merge в | `master` **И** `develop` (или `release`, если существует) |

> | Именование | `hotfix/*` |

> | Удаляется | после merge |

> | Версия | инкремент patch (`1.2.0` → `1.2.1`) |

  

### Визуальная схема GitFlow

  

```

master o----------o-----------o-----------o

|v1.0 |v1.1 |v1.2 |v1.2.1

| | | |

| | | hotfix/1.2.1

| | | |

| | release/1.2 |

| | | |

develop o---o---o--o---o---o---o---o---o---o---o

| | | | | | |

| | | | | | feature/C

| | | | feature/B |

| | feature/A | |

| | | | |

```

  

> [!tip] Легенда

> - Горизонтальные линии — постоянные ветки (`master`, `develop`)

> - Диагональные линии — временные ветки (`feature`, `release`, `hotfix`)

> - Точки пересечения — merge коммиты

> - Теги (`v1.0`, `v1.1`) — версии релизов

  

---

  

## Как реализовать GitFlow

  

### Инициализация репозитория

  

**1. Создание базовых веток**

  

```bash

mkdir myproject

cd myproject

git init

  

echo "# MyProject" > README.md

git add README.md

git commit -m "Initial commit"

  

git branch develop

git push -u origin master develop

```

  

**2. Настройка защиты веток**

  

> [!example] GitHub

> ```

> Settings → Branches → Add rule

>

> Branch name pattern: master

> ☑ Require pull request reviews before merging

> ☑ Require status checks to pass

> ☑ Include administrators

> ```

  

> [!example] GitLab

> ```

> Settings → Repository → Protected Branches

>

> master: Allowed to merge: Maintainers

> Allowed to push: No one

> develop: Allowed to merge: Developers

> Allowed to push: No one

> ```

  

### Использование Git Flow CLI

  

**Установка:**

  

```bash

# macOS

brew install git-flow

  

# Linux (Ubuntu/Debian)

apt-get install git-flow

  

# Windows

# Скачать с https://github.com/nvie/gitflow/wiki/Windows

```

  

**Инициализация:**

  

```bash

git flow init

  

# Интерактивные вопросы:

# Production branch: master

# Development branch: develop

# Feature prefix: feature/

# Release prefix: release/

# Hotfix prefix: hotfix/

# Version tag prefix: v

```

  

Автоматически с настройками по умолчанию:

  

```bash

git flow init -d

```

  

### Работа с `feature` ветками

  

**Начало разработки:**

  

```bash

# С git-flow

git flow feature start user-authentication

  

# Без git-flow

git checkout develop

git checkout -b feature/user-authentication

```

  

**Разработка:**

  

```bash

vim src/auth.js

git add src/auth.js

git commit -m "Add user authentication logic"

  

vim tests/auth.test.js

git add tests/auth.test.js

git commit -m "Add authentication tests"

  

git push -u origin feature/user-authentication

```

  

**Обновление от develop (при долгой разработке):**

  

```bash

git checkout develop

git pull origin develop

  

git checkout feature/user-authentication

git merge develop

  

# Или rebase (для более чистой истории)

git rebase develop

```

  

**Завершение фичи:**

  

```bash

# С git-flow

git flow feature finish user-authentication

  

# Без git-flow

git checkout develop

git merge --no-ff feature/user-authentication

git branch -d feature/user-authentication

git push origin develop

git push origin :feature/user-authentication # удалить удалённую ветку

```

  

> [!info] Флаг `--no-ff` (no fast-forward)

> ```

> С --no-ff:

> develop o---o---o-------o

> \ /

> feature o---o---o

>

> Без --no-ff (fast-forward):

> develop o---o---o---o---o---o

> ```

> `--no-ff` создаёт merge commit, сохраняя историю ветки. **Рекомендуется** для GitFlow.

  

### Работа с `release` ветками

  

**Создание:**

  

```bash

# С git-flow

git flow release start 1.2.0

  

# Без git-flow

git checkout develop

git checkout -b release/1.2.0

```

  

**Подготовка релиза:**

  

```bash

# Обновление версии

echo "1.2.0" > VERSION

sed -i 's/"version": ".*"/"version": "1.2.0"/' package.json

git commit -a -m "Bump version to 1.2.0"

  

# Обновление changelog

# ... редактирование CHANGELOG.md ...

  

git add CHANGELOG.md

git commit -m "Update changelog for 1.2.0"

```

  

**Тестирование и bugfix:**

  

```bash

git commit -a -m "Fix: Button alignment on mobile"

git commit -a -m "Fix: Validation error messages"

# НЕ добавлять новые фичи!

```

  

**Завершение релиза:**

  

```bash

# С git-flow

git flow release finish 1.2.0

  

# Без git-flow

git checkout master

git merge --no-ff release/1.2.0

git tag -a v1.2.0 -m "Release version 1.2.0"

  

git checkout develop

git merge --no-ff release/1.2.0

  

git branch -d release/1.2.0

git push origin master develop --tags

```

  

**Deploy:**

  

```bash

# После push в master, CI/CD автоматически деплоит на production

# Или вручную:

git checkout v1.2.0

./deploy.sh production

```

  

### Работа с `hotfix` ветками

  

> [!danger] Сценарий

> Критический баг в production (версия 1.2.0)

  

```bash

# С git-flow

git flow hotfix start 1.2.1

  

# Без git-flow

git checkout master

git checkout -b hotfix/1.2.1

```

  

**Исправление:**

  

```bash

vim src/payment.js

git commit -a -m "Fix: Critical payment processing bug"

  

echo "1.2.1" > VERSION

sed -i 's/"version": "1.2.0"/"version": "1.2.1"/' package.json

git commit -a -m "Bump version to 1.2.1"

  

npm test

```

  

**Завершение hotfix:**

  

```bash

# С git-flow

git flow hotfix finish 1.2.1

  

# Без git-flow

git checkout master

git merge --no-ff hotfix/1.2.1

git tag -a v1.2.1 -m "Hotfix version 1.2.1"

  

git checkout develop

git merge --no-ff hotfix/1.2.1

  

# Если существует release ветка — merge в неё вместо develop

git checkout release/1.3.0

git merge --no-ff hotfix/1.2.1

  

git branch -d hotfix/1.2.1

git push origin master develop --tags

```

  

### Полный workflow команды

  

> [!example] Developer 1 — Новая фича

> ```bash

> git flow feature start payment-integration

> # ... разработка ...

> git push origin feature/payment-integration

> # ... code review через Pull Request ...

> git flow feature finish payment-integration

> ```

  

> [!example] Developer 2 — Другая фича (параллельно)

> ```bash

> git flow feature start email-notifications

> # ... разработка ...

> git flow feature finish email-notifications

> ```

  

> [!example] Release Manager — Подготовка релиза

> ```bash

> # Все фичи завершены и в develop

> git flow release start 2.0.0

> # ... тестирование, bugfix ...

> git flow release finish 2.0.0

> ```

  

> [!example] Developer 3 — Hotfix (во время релиза)

> ```bash

> # Критический баг в production

> git flow hotfix start 1.2.1

> # ... исправление ...

> git flow hotfix finish 1.2.1

> ```

  

---

  

## Настройка CI/CD для GitFlow

  

### GitHub Actions

  

```yaml

# .github/workflows/gitflow.yml

name: GitFlow CI/CD

  

on:

push:

branches:

- master

- develop

- 'feature/**'

- 'release/**'

- 'hotfix/**'

pull_request:

branches:

- develop

- master

  

jobs:

test:

runs-on: ubuntu-latest

steps:

- uses: actions/checkout@v3

- uses: actions/setup-node@v3

with:

node-version: '18'

- run: npm ci

- run: npm test

- run: npm run lint

  

deploy-dev:

needs: test

if: github.ref == 'refs/heads/develop'

runs-on: ubuntu-latest

steps:

- uses: actions/checkout@v3

- run: ./deploy.sh dev

  

deploy-staging:

needs: test

if: startsWith(github.ref, 'refs/heads/release/')

runs-on: ubuntu-latest

steps:

- uses: actions/checkout@v3

- run: ./deploy.sh staging

  

deploy-production:

needs: test

if: github.ref == 'refs/heads/master'

runs-on: ubuntu-latest

steps:

- uses: actions/checkout@v3

- run: ./deploy.sh production

```

  

### GitLab CI

  

```yaml

# .gitlab-ci.yml

stages:

- test

- deploy

  

test:

stage: test

image: node:18

script:

- npm ci

- npm test

- npm run lint

coverage: '/Lines\s*:\s*(\d+\.\d+)%/'

  

deploy_dev:

stage: deploy

only:

- develop

script:

- ./deploy.sh dev

environment:

name: development

url: https://dev.myapp.com

  

deploy_staging:

stage: deploy

only:

- /^release\/.*/

script:

- ./deploy.sh staging

environment:

name: staging

url: https://staging.myapp.com

  

deploy_production:

stage: deploy

only:

- master

script:

- ./deploy.sh production

environment:

name: production

url: https://myapp.com

when: manual

```

  

---

  

## Резолвинг конфликтов с помощью IDE

  

### Почему возникают конфликты

  

```

develop: A---B---C

\

feature: D---E---F

  

Файл изменён и в C (develop) и в E (feature)

При merge develop в feature → CONFLICT

```

  

### VS Code

  

При конфликте VS Code автоматически показывает:

  

```javascript

<<<<<<< HEAD (Current Change)

function calculateTotal(items) {

return items.reduce((sum, item) => sum + item.price, 0);

}

=======

function calculateTotal(items) {

const total = items.reduce((sum, item) => sum + (item.price * item.quantity), 0);

return total;

}

>>>>>>> develop (Incoming Change)

```

  

> [!tip] Действия в VS Code

> - **Accept Current Change** — оставить свой код (из feature)

> - **Accept Incoming Change** — взять код из develop

> - **Accept Both Changes** — оставить оба варианта

> - **Compare Changes** — открыть diff editor для детального сравнения

  

**Пошаговое разрешение:**

  

```bash

git checkout feature/my-feature

git merge develop

# VS Code покажет конфликты в Source Control panel

# Открыть файл → выбрать вариант → отредактировать

git add .

git commit -m "Merge develop into feature/my-feature"

```

  

> [!tip] Полезные расширения VS Code

> - **GitLens** — продвинутая интеграция Git

> - **Git Graph** — визуализация истории веток

> - **Merge Conflict** — улучшенный интерфейс для конфликтов

  

### PhpStorm / WebStorm / IntelliJ IDEA

  

> [!note] Three-way merge

> PhpStorm показывает три панели:

> ```

> ┌──────────────┬──────────────┬──────────────┐

> │ Your │ Result │ Their │

> │ Changes │ (Middle) │ Changes │

> │ (Feature) │ │ (Develop) │

> └──────────────┴──────────────┴──────────────┘

> ```

> - `<<` — Accept Left (ваши изменения)

> - `>>` — Accept Right (их изменения)

> - `X` — Reject change

> - Middle panel — ручное редактирование результата

  

**Запуск:** `VCS → Git → Resolve Conflicts` → Click "Merge"

  

### GitKraken

  

Графический интерфейс с визуализацией веток. При конфликте файл помечен оранжевым, click → "Open Merge Tool".

  

### Sourcetree

  

`Right-click → Resolve Conflicts → Launch External Merge Tool`

  

Можно настроить внешний tool: `Preferences → Diff → External Diff Tool` (Beyond Compare, KDiff3, P4Merge, etc.)

  

### Лучшие практики разрешения конфликтов

  

> [!success] Рекомендации

> 1. **Частые merge из develop** — чем чаще обновляете feature ветку, тем меньше конфликтов

> 2. **Rebase для чистой истории** (опционально) — `git rebase develop` вместо merge

> 3. **Общайтесь с командой** — предупредите в Slack/Teams если меняете код, который может конфликтовать

> 4. **Маленькие коммиты** — легче разрешать конфликты в маленьких частях

> 5. **`.gitattributes` для merge strategies** — `package-lock.json merge=ours`

> 6. **Тестируйте после разрешения** — `npm test && npm run lint`

  

---

  

## Автоматический merge на сервере

  

### GitHub — Pull Requests

  

**Создание PR из feature ветки:**

  

```bash

git checkout -b feature/user-auth

# ... commits ...

git push -u origin feature/user-auth

```

  

На GitHub: `Compare & pull request` → заполнить детали → `Create pull request`

  

**Branch Protection Rules:**

  

```

Settings → Branches → Branch protection rules → Add rule

  

Branch name pattern: develop

☑ Require a pull request before merging

☑ Require approvals: 2

☑ Dismiss stale approvals when new commits pushed

☑ Require status checks to pass

☑ CI / test

☑ CI / lint

☑ Require conversation resolution

☑ Include administrators

```

  

### GitLab — Merge Requests

  

```bash

git checkout -b feature/user-auth

# ... commits ...

git push -u origin feature/user-auth

```

  

GitLab автоматически предложит создать MR.

  

**Merge Request Approvals:**

  

```

Settings → General → Merge request approvals

  

Approvals required: 2

☑ Prevent approval by author

☑ Require new approvals when new commits added

```

  

**Merge when pipeline succeeds:**

  

> [!info] В GitLab UI

> `Merge Request → ☑ Merge when pipeline succeeds`

> Автоматический merge когда: все approvals получены, pipeline успешен, все discussions разрешены.

  

### Автоматизация с Mergify

  

```yaml

# .mergify.yml

pull_request_rules:

- name: Auto-merge approved PRs to develop

conditions:

- base=develop

- "#approved-reviews-by>=2"

- status-success=CI / test

- label=auto-merge

- -label=do-not-merge

- -draft

actions:

merge:

method: squash

commit_message: title+body

delete_head_branch: {}

  

- name: Auto-merge hotfixes to master

conditions:

- base=master

- head~=^hotfix/

- "#approved-reviews-by>=1"

- status-success=CI / test

actions:

merge:

method: merge

delete_head_branch: {}

  

- name: Auto-update feature branches

conditions:

- base=develop

- head~=^feature/

- -conflict

actions:

update:

method: merge

```

  

---

  

## Преимущества и недостатки GitFlow

  

> [!success] Преимущества

> - **Чёткая структура** — каждая ветка имеет определённую цель; понятный процесс для всей команды; легко онбордить новых разработчиков

> - **Параллельная разработка** — множество фич одновременно; hotfix не блокируют новую разработку; подготовка релиза независима от feature-разработки

> - **Стабильность продакшна** — master всегда production-ready; тестирование в release ветках; быстрые hotfix без риска

> - **Контроль качества** — code review перед merge; CI/CD проверки; QA тестирование в release ветках

> - **История версий** — теги для каждого релиза; легко откатиться; понятный changelog

  

> [!failure] Недостатки

> - **Сложность** — много веток и правил; overhead для маленьких команд; требует дисциплины

> - **Длинный цикл релиза** — от feature до production может пройти много времени; не подходит для continuous deployment

> - **Merge hell** — много merge коммитов; сложная история Git; конфликты при долгих feature ветках

> - **Не для всех проектов** — overkill для простых проектов; не подходит для SaaS с continuous deployment; сложно для open-source

  

### Альтернативы

  

| Модель | Описание | Подходит для |

|--------|----------|-------------|

| [[GitHub Flow]] | Все merge в `master` через PR | Простые проекты, continuous deployment |

| [[Trunk-Based Development]] | Короткие feature ветки или прямо в trunk | Маленькие команды, CI/CD |

| [[GitLab Flow]] | `master` → `pre-production` → `production` | Environment-based деплой |

  

---

  

## Заключение

  

GitFlow — мощная модель ветвления, которая организует параллельную работу, обеспечивает стабильность продакшна, упрощает управление релизами и позволяет быстро исправлять критические баги.

  

> [!warning] Когда использовать GitFlow

> - Продукты с регулярными релизами (раз в 2–4 недели)

> - Команды от 5+ разработчиков

> - Проекты с высокими требованиями к качеству

> - Приложения с версионированием (mobile apps, desktop software)

  

> [!tip] Когда рассмотреть альтернативы

> - Маленькие проекты и команды (1–3 человека)

> - SaaS с continuous deployment (несколько деплоев в день)

> - Open-source проекты с внешними контрибьюторами

> - Прототипы и MVP