---

tags:

- networking

- http

- rest

- web

aliases:

- HTTP методы

- REST API

- HTTP requests

---

  

# HTTP-методы

  

> [!abstract] Обзор

> HTTP-метод определяет, какое действие нужно выполнить с ресурсом. Основные методы: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`. Каждый метод имеет свои свойства: идемпотентность, безопасность, кэшируемость.

  

## Содержание

  

- [[#Основные HTTP-методы]]

- [[#GET — Получение данных]]

- [[#POST — Создание ресурса]]

- [[#GET vs POST — Сравнение]]

- [[#Идемпотентность]]

- [[#POST vs PUT]]

- [[#PATCH — Частичное обновление]]

- [[#Stateless]]

- [[#Все HTTP-методы]]

- [[#Коды ответов]]

- [[#Шпаргалка]]

  

---

  

## Основные HTTP-методы

  

| Метод | Назначение | Пример действия |

|-------|-----------|-----------------|

| `GET` | Получить данные | Прочитать статью, получить список товаров |

| `POST` | Создать новый ресурс | Создать пользователя, отправить форму |

| `PUT` | Обновить/заменить ресурс | Обновить профиль пользователя |

| `PATCH` | Частично обновить ресурс | Изменить только email |

| `DELETE` | Удалить ресурс | Удалить пост, удалить аккаунт |

| `HEAD` | Получить заголовки (без тела) | Проверить существование файла |

| `OPTIONS` | Узнать доступные методы | CORS preflight запрос |

  

---

  

## GET — Получение данных

  

> [!info] Определение

> `GET` используется для **чтения** данных. Не должен изменять состояние сервера.

  

### Характеристики GET

  

> [!success] Преимущества

> - Только получение данных (чтение)

> - Параметры в URL (query string)

> - Можно кэшировать

> - Можно добавить в закладки

> - Виден в истории браузера

  

> [!warning] Ограничения

> - Ограничение на длину URL (~2048 символов)

> - Небезопасно для чувствительных данных (видно в логах)

  

### Примеры GET

  

```http

# Получить список всех пользователей

GET /api/users HTTP/1.1

Host: example.com

  

# Ответ:

HTTP/1.1 200 OK

Content-Type: application/json

  

[

{"id": 1, "name": "Alice", "email": "alice@example.com"},

{"id": 2, "name": "Bob", "email": "bob@example.com"}

]

```

  

```http

# Получить конкретного пользователя

GET /api/users/123 HTTP/1.1

Host: example.com

  

# Ответ:

HTTP/1.1 200 OK

Content-Type: application/json

  

{

"id": 123,

"name": "Alice",

"email": "alice@example.com",

"created_at": "2024-01-15T10:30:00Z"

}

```

  

```http

# GET с параметрами (query string)

GET /api/products?category=laptops&price_max=2000&sort=price HTTP/1.1

Host: example.com

  

# Ответ:

HTTP/1.1 200 OK

Content-Type: application/json

  

{

"total": 15,

"products": [

{"id": 1, "name": "Laptop A", "price": 1200},

{"id": 2, "name": "Laptop B", "price": 1500}

]

}

```

  

### GET в браузере

  

```javascript

// JavaScript fetch

fetch('https://example.com/api/users/123')

.then(response => response.json())

.then(data => console.log(data));

  

// Axios

axios.get('/api/users/123')

.then(response => console.log(response.data));

```

  

### GET в curl

  

```bash

# Простой GET

curl https://example.com/api/users/123

  

# С заголовками

curl -H "Authorization: Bearer token123" \

https://example.com/api/users/123

  

# С параметрами

curl "https://example.com/api/products?category=laptops&sort=price"

  

# Verbose (показать запрос и ответ)

curl -v https://example.com/api/users/123

```

  

---

  

## POST — Создание ресурса

  

> [!info] Определение

> `POST` используется для **создания** новых ресурсов. Изменяет состояние сервера.

  

### Характеристики POST

  

> [!success] Преимущества

> - Создание новых ресурсов

> - Данные в теле запроса (request body)

> - Нет ограничения на размер данных

> - Безопаснее для чувствительных данных

  

> [!warning] Ограничения

> - Не кэшируется

> - Нельзя добавить в закладки

> - Не идемпотентен (повтор создаст дубликат)

  

### Примеры POST

  

```http

# Создать нового пользователя

POST /api/users HTTP/1.1

Host: example.com

Content-Type: application/json

Content-Length: 85

  

{

"name": "Charlie",

"email": "charlie@example.com",

"password": "securepass123"

}

  

# Ответ:

HTTP/1.1 201 Created

Location: /api/users/124

Content-Type: application/json

  

{

"id": 124,

"name": "Charlie",

"email": "charlie@example.com",

"created_at": "2024-10-26T15:30:45Z"

}

```

  

```http

# Отправить форму (form data)

POST /api/login HTTP/1.1

Host: example.com

Content-Type: application/x-www-form-urlencoded

  

username=alice&password=secret123

  

# Ответ:

HTTP/1.1 200 OK

Content-Type: application/json

  

{

"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",

"user_id": 123

}

```

  

```http

# Загрузить файл (multipart)

POST /api/upload HTTP/1.1

Host: example.com

Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

  

------WebKitFormBoundary7MA4YWxkTrZu0gW

Content-Disposition: form-data; name="file"; filename="image.jpg"

Content-Type: image/jpeg

  

(binary data)

------WebKitFormBoundary7MA4YWxkTrZu0gW--

  

# Ответ:

HTTP/1.1 201 Created

Content-Type: application/json

  

{

"file_id": "abc123",

"url": "https://cdn.example.com/images/abc123.jpg"

}

```

  

### POST в браузере

  

```html

<!-- HTML форма — это POST запрос -->

<form action="/api/users" method="POST">

<input type="text" name="name" placeholder="Name">

<input type="email" name="email" placeholder="Email">

<button type="submit">Создать</button>

</form>

```

  

```javascript

// JavaScript fetch

fetch('https://example.com/api/users', {

method: 'POST',

headers: { 'Content-Type': 'application/json' },

body: JSON.stringify({

name: 'Charlie',

email: 'charlie@example.com'

})

})

.then(response => response.json())

.then(data => console.log(data));

  

// Axios

axios.post('/api/users', {

name: 'Charlie',

email: 'charlie@example.com'

})

.then(response => console.log(response.data));

```

  

### POST в curl

  

```bash

# POST с JSON данными

curl -X POST https://example.com/api/users \

-H "Content-Type: application/json" \

-d '{"name":"Charlie","email":"charlie@example.com"}'

  

# POST с form data

curl -X POST https://example.com/api/login \

-d "username=alice" \

-d "password=secret123"

  

# POST с файлом

curl -X POST https://example.com/api/upload \

-F "file=@image.jpg"

  

# POST с авторизацией

curl -X POST https://example.com/api/posts \

-H "Authorization: Bearer token123" \

-H "Content-Type: application/json" \

-d '{"title":"My post","content":"Content here"}'

```

  

---

  

## GET vs POST — Сравнение

  

| Характеристика | `GET` | `POST` |

|---------------|-------|--------|

| **Назначение** | Получить данные | Создать ресурс |

| **Данные** | В URL (query string) | В теле запроса (body) |

| **Кэширование** | Да | Нет |

| **Закладки** | Да | Нет |

| **История браузера** | Да | Нет |

| **Ограничение размера** | ~2048 символов (URL) | Нет ограничений |

| **Безопасность** | Параметры видны в URL | Данные в теле (безопаснее) |

| **Идемпотентность** | Да | Нет |

| **Изменение данных** | Не должен | Да |

  

> [!tip] Когда использовать GET

> - Получить список товаров

> - Открыть статью блога

> - Поиск по сайту

> - Пагинация (страница 1, 2, 3)

> - Фильтрация и сортировка

> - Получить профиль пользователя

  

> [!tip] Когда использовать POST

> - Регистрация нового пользователя

> - Создать новый пост в блоге

> - Отправить сообщение

> - Загрузить файл

> - Оформить заказ

> - Авторизация (логин)

  

---

  

## Идемпотентность

  

> [!important] Определение

> **Идемпотентность** — свойство операции, при котором повторное выполнение даёт тот же результат, что и однократное.

>

> Если выполнить запрос один раз или 10 раз подряд — результат будет одинаковый.

  

### Аналогия из жизни

  

```

ИДЕМПОТЕНТНО:

- Включить свет (если свет уже включён, ничего не изменится)

- Установить температуру на 20°C (повторная установка не изменит результат)

- Положить книгу на полку (если книга уже там, ничего не изменится)

  

НЕ ИДЕМПОТЕНТНО:

- Нажать кнопку "+" (каждое нажатие добавляет 1)

- Отправить SMS (каждая отправка = новое SMS)

- Положить деньги в копилку (каждый раз сумма увеличивается)

```

  

### Идемпотентность HTTP-методов

  

| Метод | Идемпотентный? | Объяснение |

|-------|----------------|-----------|

| `GET` | Да | Чтение не меняет данные. Можно читать 1000 раз — данные те же. |

| `PUT` | Да | Замена ресурса. Заменить 1 раз или 10 раз — результат одинаковый. |

| `DELETE` | Да | Удалить ресурс. Если удалить дважды — второй раз уже нечего удалять (404). |

| `PATCH` | Может быть | Зависит от реализации. |

| `POST` | Нет | Каждый запрос создаёт новый ресурс. |

  

### Примеры идемпотентности

  

#### GET — Идемпотентный

  

```http

# Первый запрос

GET /api/users/123 HTTP/1.1

# Ответ: {"id": 123, "name": "Alice", "email": "alice@example.com"}

  

# Второй запрос (такой же)

GET /api/users/123 HTTP/1.1

# Ответ (такой же): {"id": 123, "name": "Alice", "email": "alice@example.com"}

  

# Данные НЕ изменились!

```

  

#### PUT — Идемпотентный

  

```http

# Первый запрос: обновить email

PUT /api/users/123 HTTP/1.1

Content-Type: application/json

{"name": "Alice", "email": "alice_new@example.com"}

  

# Ответ: {"id": 123, "name": "Alice", "email": "alice_new@example.com"}

  

# Второй запрос (точно такой же)

PUT /api/users/123 HTTP/1.1

Content-Type: application/json

{"name": "Alice", "email": "alice_new@example.com"}

  

# Ответ (такой же): {"id": 123, "name": "Alice", "email": "alice_new@example.com"}

  

# Пользователь обновлён до того же состояния.

```

  

#### DELETE — Идемпотентный

  

```http

# Первый запрос: удалить пользователя

DELETE /api/users/123 HTTP/1.1

# Ответ: HTTP/1.1 204 No Content

  

# Второй запрос (попытка удалить снова)

DELETE /api/users/123 HTTP/1.1

# Ответ: HTTP/1.1 404 Not Found

  

# Результат тот же: пользователь не существует!

```

  

#### POST — НЕ идемпотентный

  

```http

# Первый запрос: создать пользователя

POST /api/users HTTP/1.1

Content-Type: application/json

{"name": "Bob", "email": "bob@example.com"}

  

# Ответ: HTTP/1.1 201 Created

# {"id": 124, "name": "Bob", "email": "bob@example.com"}

  

# Второй запрос (точно такой же)

POST /api/users HTTP/1.1

Content-Type: application/json

{"name": "Bob", "email": "bob@example.com"}

  

# Ответ: HTTP/1.1 201 Created

# {"id": 125, "name": "Bob", "email": "bob@example.com"}

  

# СОЗДАН НОВЫЙ ПОЛЬЗОВАТЕЛЬ! Теперь в БД два пользователя Bob (ID 124 и 125)

```

  

### Зачем нужна идемпотентность

  

> [!example] Надёжность при сетевых ошибках

> 1. Клиент отправляет `PUT` запрос

> 2. Сервер обработал и обновил данные

> 3. Но ответ потерялся из-за сбоя сети

> 4. Клиент не получил ответ и думает, что запрос не выполнен

> 5. Клиент отправляет запрос **повторно**

>

> С `PUT` — повторный запрос безопасен, данные те же.

> С `POST` — повторный запрос создаст **дубликат**.

  

```javascript

// Можно безопасно повторить GET/PUT/DELETE

async function updateUser(userId, data) {

try {

const response = await fetch(`/api/users/${userId}`, {

method: 'PUT',

body: JSON.stringify(data)

});

return response.json();

} catch (error) {

// Безопасно повторить — результат будет тот же

return updateUser(userId, data);

}

}

  

// С POST нужна осторожность

async function createUser(data) {

try {

const response = await fetch('/api/users', {

method: 'POST',

body: JSON.stringify(data)

});

return response.json();

} catch (error) {

// НЕ повторяем автоматически! Иначе создадим дубликат

console.error('Failed to create user');

}

}

```

  

---

  

## POST vs PUT

  

### Основное различие

  

| Характеристика | `POST` | `PUT` |

|---------------|--------|-------|

| **Цель** | Создать новый ресурс | Обновить/заменить существующий |

| **ID ресурса** | Сервер генерирует ID | Клиент знает ID |

| **URL** | Коллекция: `/api/users` | Конкретный ресурс: `/api/users/123` |

| **Идемпотентность** | Нет | Да |

| **Повторный запрос** | Создаст дубликат | Обновит до того же состояния |

| **Код ответа** | `201 Created` | `200 OK` или `204 No Content` |

  

### POST — Создание

  

```http

POST /api/users HTTP/1.1

Host: example.com

Content-Type: application/json

  

{

"name": "David",

"email": "david@example.com"

}

  

# Ответ:

HTTP/1.1 201 Created

Location: /api/users/126

  

{

"id": 126,

"name": "David",

"email": "david@example.com",

"created_at": "2024-10-26T16:00:00Z"

}

  

# Повторный POST → создаст НОВОГО пользователя (ID 127)

```

  

> [!tip] Когда использовать POST

> - Клиент **НЕ** знает ID нового ресурса

> - ID генерирует сервер

> - Создание нового ресурса в коллекции

  

### PUT — Замена

  

```http

PUT /api/users/126 HTTP/1.1

Host: example.com

Content-Type: application/json

  

{

"name": "David Updated",

"email": "david_new@example.com"

}

  

# Ответ:

HTTP/1.1 200 OK

  

{

"id": 126,

"name": "David Updated",

"email": "david_new@example.com",

"updated_at": "2024-10-26T16:05:00Z"

}

  

# Повторный PUT → обновит до того же состояния

```

  

> [!tip] Когда использовать PUT

> - Клиент знает ID ресурса

> - Замена **всего** ресурса целиком

> - Обновление существующего ресурса

  

### PUT для создания (если ID известен)

  

```http

PUT /api/users/999 HTTP/1.1

Content-Type: application/json

  

{

"name": "Eve",

"email": "eve@example.com"

}

  

# Если пользователь 999 НЕ существует:

HTTP/1.1 201 Created

{"id": 999, "name": "Eve", "email": "eve@example.com"}

  

# Если пользователь 999 УЖЕ существует:

HTTP/1.1 200 OK

{"id": 999, "name": "Eve", "email": "eve@example.com"}

```

  

---

  

## PATCH — Частичное обновление

  

> [!info] Определение

> `PATCH` обновляет только **часть** ресурса (в отличие от `PUT`, который заменяет целиком).

  

```http

PATCH /api/users/126 HTTP/1.1

Content-Type: application/json

  

{

"email": "david_newest@example.com"

}

  

# Ответ:

HTTP/1.1 200 OK

{

"id": 126,

"name": "David Updated",

"email": "david_newest@example.com"

}

```

  

### Разница PUT vs PATCH

  

```http

# Исходные данные:

{

"id": 126,

"name": "David",

"email": "david@example.com",

"age": 30,

"city": "Moscow"

}

  

# PUT: заменить ВЕСЬ ресурс

PUT /api/users/126

{"name": "David", "email": "new@example.com"}

  

# Результат:

{

"id": 126,

"name": "David",

"email": "new@example.com",

"age": null,

"city": null

}

  

# PATCH: обновить только email

PATCH /api/users/126

{"email": "new@example.com"}

  

# Результат:

{

"id": 126,

"name": "David",

"email": "new@example.com",

"age": 30,

"city": "Moscow"

}

```

  

### Примеры использования

  

```javascript

// POST — создать нового (ID неизвестен)

const response = await fetch('/api/users', {

method: 'POST',

headers: { 'Content-Type': 'application/json' },

body: JSON.stringify({

name: 'Alice',

email: 'alice@example.com'

})

});

const user = await response.json();

console.log(user.id); // 123 (сервер вернул ID)

  

// PUT — заменить пользователя (ID известен)

const response = await fetch('/api/users/123', {

method: 'PUT',

headers: { 'Content-Type': 'application/json' },

body: JSON.stringify({

name: 'Alice Updated',

email: 'alice_new@example.com',

age: 25,

city: 'London'

})

});

  

// PATCH — изменить только email (ID известен)

const response = await fetch('/api/users/123', {

method: 'PATCH',

headers: { 'Content-Type': 'application/json' },

body: JSON.stringify({

email: 'alice_newest@example.com'

})

});

```

  

---

  

## Stateless

  

> [!important] Определение

> **Stateless (без состояния)** — каждый HTTP-запрос независим и содержит всю необходимую информацию для выполнения.

>

> Сервер **не хранит** информацию о предыдущих запросах клиента. Каждый запрос — это «новое знакомство».

  

### Аналогия из жизни

  

```

Stateful (с состоянием) — Банк:

  

Вы: "Здравствуйте, я хочу открыть счёт"

Банк: "Хорошо, ваш счёт №12345. Запомнил вас."

  

(на следующий день)

Вы: "Хочу положить 1000₽"

Банк: "Отлично! Вы клиент №12345. Кладу на ваш счёт."

↑ Банк ПОМНИТ вас

  

---

  

Stateless (без состояния) — Почта:

  

Вы: "Отправьте письмо в Москву"

Почта: "Кто вы? Куда отправлять? Сколько платить?"

  

Вы: "Вот все данные: от кого, куда, марка"

Почта: "Отправляю"

  

(на следующий день)

Вы: "Отправьте ещё письмо"

Почта: "Кто вы? Куда? Сколько?"

↑ Почта НЕ ПОМНИТ вас (каждый раз нужны все данные)

```

  

### HTTP — Stateless протокол

  

```http

# Первый запрос

GET /api/products HTTP/1.1

Host: example.com

Authorization: Bearer token123

# Ответ: HTTP/1.1 200 OK — [список товаров]

  

# Второй запрос (от того же клиента)

GET /api/cart HTTP/1.1

Host: example.com

Authorization: Bearer token123 ← Нужно СНОВА отправить токен!

  

# Сервер НЕ ПОМНИТ, что это тот же клиент!

# Каждый запрос должен содержать всю информацию.

```

  

### Как хранить состояние в Stateless HTTP

  

#### Токены авторизации (JWT, Bearer)

  

```http

GET /api/profile HTTP/1.1

Host: example.com

Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

  

# Сервер расшифровывает токен → узнает кто пользователь

# Но сам токен хранится у КЛИЕНТА, не на сервере!

```

  

#### Cookies

  

```http

# Первый запрос: логин

POST /api/login HTTP/1.1

Content-Type: application/json

{"username": "alice", "password": "secret"}

  

# Ответ: сервер отправляет cookie

HTTP/1.1 200 OK

Set-Cookie: session_id=abc123; HttpOnly; Secure

  

# Последующие запросы: браузер автоматически отправляет cookie

GET /api/profile HTTP/1.1

Cookie: session_id=abc123

```

  

### Преимущества и недостатки Stateless

  

> [!success] Преимущества

> - **Масштабируемость** — запросы можно направлять на любой сервер, состояние не привязано

> - **Надёжность** — если сервер упал, клиент переотправит запрос на другой

> - **Простота** — сервер не тратит память на хранение состояний

  

> [!warning] Недостатки

> - **Больше данных** — нужно каждый раз отправлять токен/cookie

> - **Сложность на клиенте** — клиент должен хранить и отправлять токен с каждым запросом

  

### Stateless в REST API

  

```javascript

const token = localStorage.getItem('auth_token');

  

async function getProducts() {

const response = await fetch('/api/products', {

headers: { 'Authorization': `Bearer ${token}` }

});

return response.json();

}

  

async function addToCart(productId) {

const response = await fetch('/api/cart', {

method: 'POST',

headers: {

'Authorization': `Bearer ${token}`,

'Content-Type': 'application/json'

},

body: JSON.stringify({ product_id: productId })

});

return response.json();

}

  

async function checkout() {

const response = await fetch('/api/checkout', {

method: 'POST',

headers: { 'Authorization': `Bearer ${token}` }

});

return response.json();

}

  

// Сервер НЕ ПОМНИТ, что это один и тот же пользователь

// Токен приходит в каждом запросе и сервер его проверяет

```

  

---

  

## Все HTTP-методы

  

### Полный список

  

| Метод | Идемпотентный | Безопасный | Назначение |

|-------|---------------|------------|-----------|

| `GET` | Да | Да | Получить ресурс |

| `POST` | Нет | Нет | Создать ресурс |

| `PUT` | Да | Нет | Заменить ресурс |

| `PATCH` | Может быть | Нет | Частично обновить |

| `DELETE` | Да | Нет | Удалить ресурс |

| `HEAD` | Да | Да | Получить заголовки (как GET, но без тела) |

| `OPTIONS` | Да | Да | Узнать доступные методы |

| `CONNECT` | Нет | Нет | Установить туннель (для прокси) |

| `TRACE` | Да | Да | Отладка (echo запрос) |

  

> [!info] Безопасный метод

> Безопасный = не изменяет данные на сервере. К безопасным относятся: `GET`, `HEAD`, `OPTIONS`, `TRACE`.

  

### HEAD — Только заголовки

  

```http

HEAD /api/users/123 HTTP/1.1

Host: example.com

  

# Ответ (БЕЗ тела, только заголовки):

HTTP/1.1 200 OK

Content-Type: application/json

Content-Length: 156

Last-Modified: Wed, 26 Oct 2024 10:30:00 GMT

ETag: "abc123"

```

  

> [!tip] Использование HEAD

> - Проверить существование ресурса (200 vs 404)

> - Узнать размер файла перед скачиванием

> - Проверить `Last-Modified` (изменился ли файл)

  

### OPTIONS — Доступные методы

  

```http

OPTIONS /api/users/123 HTTP/1.1

Host: example.com

  

# Ответ:

HTTP/1.1 200 OK

Allow: GET, PUT, PATCH, DELETE

Access-Control-Allow-Methods: GET, PUT, PATCH, DELETE

Access-Control-Allow-Origin: *

```

  

> [!tip] Использование OPTIONS

> - CORS preflight запрос (браузер автоматически отправляет перед `POST`/`PUT`)

> - Узнать какие методы поддерживает сервер

  

---

  

## Коды ответов

  

### 2xx — Успех

  

| Код | Статус | Значение |

|-----|--------|----------|

| `200` | OK | Запрос выполнен успешно |

| `201` | Created | Ресурс создан (`POST`, `PUT`) |

| `204` | No Content | Успех, но нет тела ответа (`DELETE`) |

  

### 3xx — Перенаправление

  

| Код | Статус | Значение |

|-----|--------|----------|

| `301` | Moved Permanently | Ресурс перемещён навсегда |

| `302` | Found | Временное перенаправление |

| `304` | Not Modified | Ресурс не изменился (кэш) |

  

### 4xx — Ошибка клиента

  

| Код | Статус | Значение |

|-----|--------|----------|

| `400` | Bad Request | Неверный запрос (невалидные данные) |

| `401` | Unauthorized | Требуется авторизация |

| `403` | Forbidden | Доступ запрещён |

| `404` | Not Found | Ресурс не найден |

| `405` | Method Not Allowed | Метод не поддерживается |

| `409` | Conflict | Конфликт (например, дубликат email) |

| `422` | Unprocessable Entity | Валидация не прошла |

| `429` | Too Many Requests | Слишком много запросов (rate limit) |

  

### 5xx — Ошибка сервера

  

| Код | Статус | Значение |

|-----|--------|----------|

| `500` | Internal Server Error | Внутренняя ошибка сервера |

| `502` | Bad Gateway | Ошибка прокси/шлюза |

| `503` | Service Unavailable | Сервис недоступен |

| `504` | Gateway Timeout | Таймаут прокси |

  

---

  

## Шпаргалка

  

### Методы CRUD

  

```mermaid

flowchart LR

C["`**POST**

Create`"]

R["`**GET**

Read`"]

U["`**PUT/PATCH**

Update`"]

D["`**DELETE**

Delete`"]

C --> R --> U --> D

```

  

| CRUD | HTTP-метод | URL | Body | Код ответа |

|------|-----------|-----|------|-----------|

| **C**reate | `POST /api/users` | `/api/users` | `{name, email}` | `201 Created` |

| **R**ead | `GET /api/users/123` | `/api/users/123` | — | `200 OK` |

| **U**pdate | `PUT /api/users/123` | `/api/users/123` | `{name, email}` | `200 OK` |

| **U**pdate | `PATCH /api/users/123` | `/api/users/123` | `{email}` | `200 OK` |

| **D**elete | `DELETE /api/users/123` | `/api/users/123` | — | `204 No Content` |

  

### Идемпотентность

  

| Да | Нет |

|----|-----|

| `GET` | `POST` |

| `PUT` | |

| `DELETE` | |

| `HEAD` | |

| `OPTIONS` | |

  

### PUT vs PATCH

  

```

PUT /api/users/123 → Заменяет ВЕСЬ ресурс

{"name": "A", "email": "a@b.com", "age": 25}

  

PATCH /api/users/123 → Обновляет ТОЛЬКО указанные поля

{"email": "new@b.com"}

```

  

### Безопасные vs Небезопасные

  

```

Безопасные (не меняют данные): Небезопасные (меняют данные):

GET POST

HEAD PUT

OPTIONS PATCH

TRACE DELETE

CONNECT

```

  

### Коды ответов — быстро

  

```

2xx — Всё ок → 200 OK, 201 Created, 204 No Content

3xx — Иди туда → 301 Навсегда, 302 Временно, 304 Не изменилось

4xx — Твоя ошибка → 400 Плохой запрос, 401 Не авторизован, 403 Запрещено, 404 Не найден

5xx — Моя ошибка → 500 Сервер сломался, 502 Шлюз, 503 Недоступен, 504 Таймаут

```

  

> [!warning] Запомни

> - `401` — **кто ты?** (не авторизован, нужен логин)

> - `403` — **ты не можешь** (авторизован, но доступа нет)

> - `404` — **такого нет** (ресурс не найден)

  

### Stateless — за 10 секунд

  

```

Каждый запрос = самостоятельный.

Сервер не помнит предыдущие запросы.

Всё состояние — в токене/cookie, которое шлёт клиент.

```