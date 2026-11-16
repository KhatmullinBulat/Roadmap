---
type: careers/interview
section: Браузер и сети
topic: HTTP
status: ready
created: 2026-11-16
tags: [interview, http, networking]
---

# Из чего состоит HTTP-запрос

## Краткий ответ (30 секунд)

HTTP-запрос — это текст (в HTTP/1.1; в HTTP/2/3 — бинарные фреймы) из трёх частей: **строка запроса** (`МЕТОД путь HTTP/версия`), **заголовки** (`Ключ: значение` по строке на каждый), **пустая строка** и **тело** (опционально, для POST/PUT/PATCH). Ответ устроен так же: статусная строка (`HTTP/1.1 200 OK`), заголовки, тело. Метод, URL, версия, заголовки и тело — всё, что нужно для ответа.

## Подробно

### Сырой HTTP/1.1-запрос

```http
POST /api/users?source=web HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (…)
Content-Type: application/json
Content-Length: 27
Accept: application/json
Authorization: Bearer eyJ…
Cookie: session=abc; theme=dark

{"name":"Анна","age":28}
```

Строка — `POST /api/users?source=web HTTP/1.1` (метод, **request target**, версия). Затем заголовки, пустая строка, тело.

### Компоненты

| Часть | Что содержит |
|---|---|
| Стартовая строка | метод, путь + query, версия |
| Заголовки запроса | `Host` (обязателен в 1.1), `Accept`, `Content-Type`, `Cookie`, `Authorization`, `If-None-Match` |
| Тело | JSON, form-data, файлы; может отсутствовать |

### Ответ

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Cache-Control: max-age=3600
ETag: "abc123"
Set-Cookie: session=xyz; HttpOnly; Secure; SameSite=Lax

{"id":1,"name":"Анна"}
```

- **Статус** — код + фраза (`200 OK`).
- **Заголовки ответа** — `Content-Type`, `Content-Length`, `Cache-Control`, `ETag`, `Set-Cookie`, `Access-Control-Allow-Origin`.

### Что нужно знать про метод

**Безопасные** (не меняют состояние): `GET`, `HEAD`, `OPTIONS`, `TRACE`.
**Идемпотентные** (повтор даёт тот же эффект): `GET`, `HEAD`, `PUT`, `DELETE`, `OPTIONS`.
**Не идемпотентные**: `POST`, `PATCH` (PATCH формально не обязан быть идемпотентным, хотя может).

| Метод | Назначение | Тело |
|---|---|---|
| GET | получить | нет (по спецификации) |
| POST | создать/выполнить действие | да |
| PUT | заменить целиком | да |
| PATCH | частично изменить | да |
| DELETE | удалить | обычно нет |
| HEAD | как GET, но без тела ответа | нет |
| OPTIONS | какие методы разрешены / preflight | нет |

### Статусные коды

- **1xx** — информационные; `101 Switching Protocols` (WebSocket), `103 Early Hints`.
- **2xx** — успех: `200`, `201 Created`, `204 No Content`, `206 Partial Content`.
- **3xx** — перенаправления: `301`, `302`, `304 Not Modified`, `307`, `308`.
- **4xx** — ошибка клиента: `400`, `401`, `403`, `404`, `405`, `409`, `429`.
- **5xx** — ошибка сервера: `500`, `502`, `503`, `504`.

### HTTP/2 и HTTP/3

- **HTTP/2**: бинарный протокол, мультиплексирование потоков, сжатие заголовков (HPACK), без `Connection: keep-alive` как отдельной концепции. Заголовки — фреймы `HEADERS`, тело — `DATA`.
- **HTTP/3**: поверх **QUIC** (UDP), меньше задержек установки соединения (0/1-RTT), сохраняет работу при потере пакетов.
- Смысл «что состоит из» не меняется — меняется кодировка: вместо текста на проводе бинарные фреймы и ID потоков.

### Частые ошибки в ответе

- «Метод — часть заголовков» — нет, он в стартовой строке.
- «GET может иметь тело» — формально можно отправить, но серверы могут не читать; по спецификации у GET семантика тела не определена.
- «Content-Length есть всегда» — нет, для `Transfer-Encoding: chunked` его нет.
- «Cookie — отдельный протокол» — это обычный заголовок (`Cookie`/`Set-Cookie`).

## Связанные вопросы

- [[HTTP и HTTPS]]
- [[CORS и preflight]]
- [[Cookies и флаги]]

## Ресурсы

- RFC 9110 (HTTP Semantics): https://www.rfc-editor.org/rfc/rfc9110
- MDN: https://developer.mozilla.org/ru/docs/Web/HTTP/Messages
