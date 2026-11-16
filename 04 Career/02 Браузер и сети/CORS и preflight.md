---
type: careers/interview
section: Браузер и сети
topic: Безопасность
status: ready
created: 2026-11-16
tags: [interview, browser, cors, security]
---

# CORS и preflight

## Краткий ответ (30 секунд)

**CORS (Cross-Origin Resource Sharing)** — механизм, по которому **браузер** разрешает фронтенду читать ответы с **другого origin** (схема + домен + порт). Это защита пользователя, а не сервера: без CORS любой сайт мог бы читать чужие ответы через cookie пользователя. Сервер отвечает заголовками `Access-Control-Allow-Origin` и др. Для **не-простых** запросов (методы кроме GET/HEAD/POST, `Content-Type: application/json`, кастомные заголовки) браузер сначала шлёт **preflight** — `OPTIONS` с `Access-Control-Request-Method`/`-Headers`, спрашивая разрешение, и только потом основной запрос.

## Подробно

### Что такое origin

`origin = scheme + host + port`. `https://api.site.com` и `https://site.com` — разные origin. `http://site.com` и `https://site.com` — тоже разные. `localhost:3000` и `localhost:5173` — разные.

### Простой запрос (simple request)

Не требует preflight, если:
- метод — `GET`, `HEAD` или `POST`;
- заголовки — только «безопасные» (`Accept`, `Accept-Language`, `Content-Language`, `Content-Type` с ограниченным значением);
- `Content-Type` — `text/plain`, `multipart/form-data` или `application/x-www-form-urlencoded`.

Ответ всё равно должен содержать `Access-Control-Allow-Origin`. Если его нет — браузер **не отдаст** JS тело ответа (хотя запрос дошёл до сервера!). В Network такой запрос может выглядеть как ошибка статуса 0.

### Preflight

```http
OPTIONS /api/users HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: content-type, authorization
```

Ответ сервера:

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: content-type, authorization
Access-Control-Max-Age: 600
```

- `Access-Control-Max-Age` — сколько секунд браузер кэширует preflight (по умолчанию мало, поэтому в Network он виден почти перед каждым запросом).
- Preflight не отправляется для «простых» запросов.
- Preflight всегда `OPTIONS` без тела.

### Заголовки, которые надо знать

| Заголовок | Кто шлёт | Что значит |
|---|---|---|
| `Origin` | браузер | откуда запрос, **нельзя подделать из JS** |
| `Access-Control-Allow-Origin` | сервер | кто может читать (`*` или конкретный origin) |
| `Access-Control-Allow-Methods` | сервер | разрешённые методы для preflight |
| `Access-Control-Allow-Headers` | сервер | разрешённые заголовки |
| `Access-Control-Allow-Credentials` | сервер | можно ли отправлять cookie с `credentials` |
| `Access-Control-Expose-Headers` | сервер | какие заголовки ответа видны JS (кроме простых) |
| `Access-Control-Max-Age` | сервер | кэш preflight |

### Credentials

```js
fetch("/api", { credentials: "include" });
```

- Cookie отправляются, только если клиент указал `credentials: "include"` **и** сервер ответил `Access-Control-Allow-Credentials: true`.
- При этом `Access-Control-Allow-Origin` **не может быть `*`** — нужен конкретный origin. Это классическая причина «работает без кук, ломается с куками».
- `SameSite=None` у cookie тоже нужен для cross-site.

### Частые заблуждения

1. **CORS — это браузер.** `curl` и Postman ничего не проверяют.
2. **Ошибка CORS — «сервер не пустил».** Обычно сервер просто не вернул нужные заголовки (или вернул не тот origin).
3. **Прокси в dev-сервере** убирает CORS, потому что запрос идёт с того же origin.
4. **CORS не замена CSRF-защите.** Запрос мог уйти на сервер и выполниться (для простых методов), просто ответ не читается.
5. **`*` + `credentials`** несовместимы.

### Отличие от CSRF

- **CORS** — про **чтение** ответа с чужого origin.
- **CSRF** — про **отправку** нежелательного запроса от лица пользователя. Защищают `SameSite`, CSRF-токены, проверка `Origin`/`Referer` на сервере.

> [!tip] Как отвечать
> «CORS — механизм браузера, чтобы дать JS читать ответы с другого origin. Для не-простых запросов идёт OPTIONS preflight, где сервер разрешает метод и заголовки. Cookie — только при `credentials: include` и явном origin без `*`».

## Связанные вопросы

- [[Cookies и флаги]]
- [[HTTP и HTTPS]]
- [[Из чего состоит HTTP запрос]]

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/HTTP/CORS
- Fetch spec: https://fetch.spec.whatwg.org/
