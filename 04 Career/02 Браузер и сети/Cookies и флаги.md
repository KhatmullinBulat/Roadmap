---
type: careers/interview
section: Браузер и сети
topic: Хранение
status: ready
created: 2026-11-16
tags: [interview, browser, cookies, storage]
---

# Cookies и их флаги

## Краткий ответ (30 секунд)

Cookie — небольшая пара «имя=значение», которую сервер ставит заголовком `Set-Cookie`, а браузер автоматически прикладывает к запросам на этот домен (`Cookie`). Флаги: **`HttpOnly`** — cookie недоступна из JS (защита от XSS-кражи токена); **`Secure`** — только по HTTPS; **`SameSite=Strict/Lax/None`** — когда отправлять при cross-site запросах (защита от CSRF); `Domain`, `Path`, `Max-Age`/`Expires` — область и срок жизни, `Partitioned` (CHIPS) — привязка к top-level сайту.

## Подробно

### Установка

```http
Set-Cookie: session=abc123; Max-Age=3600; Path=/; Domain=example.com; Secure; HttpOnly; SameSite=Lax
```

```js
// только для не-HttpOnly: чтение и запись из JS
document.cookie; // "theme=dark; lang=ru"
document.cookie = "theme=light; path=/; max-age=86400";
```

Из JS нельзя выставить `HttpOnly` (это бессмысленно — такой флаг ставит сервер, чтобы JS *не читал*).

### Флаги подробно

| Флаг | Что делает |
|---|---|
| `HttpOnly` | недоступна через `document.cookie`; защищает от кражи токена при XSS (но не от действий от имени пользователя) |
| `Secure` | отправляется только по HTTPS; `localhost` считается защищённым в современных браузерах |
| `SameSite=Strict` | не отправляется в cross-site запросах вообще (даже при переходе с другого сайта по ссылке) |
| `SameSite=Lax` | по умолчанию; отправляется при top-level навигации GET, не отправляется при cross-site POST/iframe/fetch |
| `SameSite=None` | отправляется всегда, обязательно с `Secure` |
| `Domain` | на какие поддомены; `example.com` → `api.example.com` тоже |
| `Path` | путь приложения (не защита, только область) |
| `Max-Age` / `Expires` | срок жизни; без них — session-cookie до закрытия браузера |
| `Partitioned` | cookie для каждой top-level страницы отдельно (CHIPS, анти-tracking) |

### Lax по умолчанию

С Chrome 80, Firefox, Safari сегодня `SameSite=Lax` по умолчанию, если флаг не указан. Это ломало старые интеграции: `iframe`-логины, cross-site POST. Для таких случаев нужен `SameSite=None; Secure`.

### Cookie и безопасность

- **XSS** — крадёт `document.cookie` (если не `HttpOnly`), читает localStorage (если не хранить там токены).
- **CSRF** — подделывает запрос от лица пользователя; помогают `SameSite=Lax/Strict`, CSRF-токены, проверка `Origin`.
- **MITM** (без HTTPS) — крадёт cookie в пути; помогает `Secure`.

### Размер и ограничения

- ~4 КБ на cookie, ~50 cookies на домен (зависит от браузера).
- Каждая cookie уходит с **каждым** запросом на домен (включая статику!) — это трафик и задержка.
- Cookie не хранит объекты — только строку; сериализуйте JSON, но следите за размером.

### Для фронтенда

Токены: `HttpOnly` cookie — лучший выбор для refresh/session. `localStorage` уязвим к XSS: токен читается любым скриптом на странице. JWT в `localStorage` — распространённый, но рискованный паттерн.

> [!tip] Формулировка
> «Cookie шлёт браузер автоматически на каждый запрос домена. `HttpOnly` исключает JS — защита от XSS, `Secure` — только HTTPS, `SameSite` — от CSRF и нежелательных cross-site запросов».

## Связанные вопросы

- [[CORS и preflight]]
- [[Где хранить данные на фронтенде]]

## Ресурсы

- MDN `Set-Cookie`: https://developer.mozilla.org/ru/docs/Web/HTTP/Headers/Set-Cookie
