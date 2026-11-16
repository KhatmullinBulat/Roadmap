---
type: careers/interview
section: Vue
topic: State
status: ready
created: 2026-11-16
tags: [interview, vue, auth, security]
---

# Хранение auth-состояния

## Краткий ответ (30 секунд)

**Access token** — короткоживущий, нужен для запросов; **refresh token** — долгоживущий, для обновления access. Безопасная схема для браузера: refresh — в **`HttpOnly; Secure; SameSite` cookie**, access — в памяти (Pinia, не в `localStorage`) на время сессии; профиль — в сторе, гидратируется из запроса `/me` или идемпотентного refresh. **Не хранить долгоживущий токен в `localStorage`** — он доступен любому JS при XSS. Для SSR (Nuxt) куки — единственный естественный источник (сервер их видит), `localStorage` — нет. Всё, что нужно, ходит через ваш backend-прокси/`server`-маршруты; сторонние API не должны получать ваш cookie.

## Подробно

### Что где

| Данные | Где | Почему |
|---|---|---|
| refresh token | `HttpOnly` cookie | недоступен JS, живёт между сессиями |
| access token | память (Pinia) | утечка только на время жизни страницы (XSS всё равно опасен, но он ephemeral) |
| профиль, роли, настройки | Pinia/кэш (или заново с `/me`) | не критичен для кражи токена |
| csrf-токен | cookie/ответ (double submit) | защита от CSRF для cookie-запросов |
| «запомнить меня» | refresh cookie с `Max-Age` | срок определяется сервером |

### Схема

```
POST /auth/login     → Set-Cookie: refresh=…; HttpOnly; Secure; SameSite=Lax
                     → JSON: { accessToken, user }
GET  /api/...        → Authorization: Bearer <access> (из памяти)
POST /auth/refresh   → по cookie → новый access (и refresh-rotation)
POST /auth/logout    → очистить cookie + погасить refresh на сервере
```

- **Refresh rotation**: каждый refresh выдаёт новый и инвалидирует старый; повтор старым = подозрение на кражу → разлогин всех сессий.
- **CSRF**: cookie отправляется автоматически → нужны `SameSite=Lax/Strict`, double-submit токен или `Origin`-проверка на сервере. `SameSite=None` — только если нужен cross-site и с `Secure`.
- **CORS**: `credentials: "include"` + `Access-Control-Allow-Credentials`, без `*` в origin.

### Как отдавать в API

Варианты:

1. **Cookie-сессия целиком** (без JWT) — простой, надёжный бэкенд; фронт не хранит токенов.
2. **JWT access + refresh cookie** — SPA-friendly.
3. **`localStorage` access + refresh** — прост, но уязвим к XSS; для внутренних/непубличных рисков допустим с оговорками.

На собеседовании лучше показать осознанность: «если хранить refresh в localStorage, XSS = украли сессию; HttpOnly cookie закрывает эту дыру, но добавляет CSRF».

### SSR (Nuxt)

- **Сервер не видит `localStorage`** — запросы на SSR к защищённым API не авторизуются.
- Cookie: сервер читает `useCookie()`/`useRequestHeaders(["cookie"])` и передаёт бэкенду.
- Middleware на сервере может редиректить до рендера.
- См. [[34 Авторизация в SSR]].

### Гидратация состояния

- Не доверять `localStorage` как источнику правды (можно подделать).
- Профиль/роли — с сервера (`/me`), refresh при загрузке приложения.
- Кэшировать только UI-подсказки, не решения о доступе.

### Выход

- Сервер: инвалидировать refresh (blacklist/версия).
- Клиент: очистить стор, cookie (сервер `Set-Cookie` с `Max-Age=0`).
- Отозвать из других вкладок — событие/BroadcastChannel.

### Ловушки

- Refresh в `localStorage` + access в cookie — путаница.
- `SameSite=None` без `Secure` — cookie не сохранится.
- Cookie на поддомене (`Domain=.example.com`) — доступна всем поддоменам.
- Логирование токенов в аналитике.

> [!warning] Что ожидает интервьюер
> «localStorage небезопасен для refresh; cookie HttpOnly — норма; access — в памяти; CSRF надо учитывать; SSR требует cookie». Это и есть полный ответ, а не «храним в localStorage, там же где всё».

## Связанные вопросы

- [[34 Авторизация в SSR]]
- [[Cookies и флаги]]
- [[21 Store и Pinia]]

## Ресурсы

- OWASP JWT: https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html
