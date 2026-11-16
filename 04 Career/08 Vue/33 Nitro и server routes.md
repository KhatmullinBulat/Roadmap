---
type: careers/interview
section: Vue
topic: Nuxt
status: ready
created: 2026-11-16
tags: [interview, nuxt, nitro, server-routes]
---

# Nitro и server routes

## Краткий ответ (30 секунд)

**Nitro** — серверный движок Nuxt (на H3): собирает универсальный сервер, который умеет отдавать SSR-HTML, API-эндпоинты, кэш, таймеры, storage. **Server routes** — файлы в `server/api/` (и `server/routes/`) превращаются в HTTP-эндпоинты по пути: `server/api/users/[id].get.ts` → `GET /api/users/:id`. Внутри — `defineEventHandler`, `readBody`, `getQuery`, `createError`, `useRuntimeConfig`, доступ к Node (`event.node.req/res`), к БД/секретам. Это «full-stack внутри Nuxt»: BFF-слой, проксирование API, защита токенов, обработка форм — без отдельного бэкенда.

## Подробно

### Файлы

```
server/
  api/
    users/index.get.ts        → GET  /api/users
    users/[id].get.ts         → GET  /api/users/:id
    users/index.post.ts       → POST /api/users
  routes/
    sitemap.xml.ts            → GET  /sitemap.xml
  middleware/auth.ts          → на каждый запрос
  plugins/                  → инициализация Nitro
  utils/                    → серверные утилиты
```

### Обработчик

```ts
// server/api/users/[id].get.ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, "id");
  if (!id) throw createError({ statusCode: 400, statusMessage: "id required" });

  const user = await db.user.findUnique({ where: { id } });
  if (!user) throw createError({ statusCode: 404, statusMessage: "Not found" });

  return user; // JSON
});
```

```ts
// POST
const body = await readValidatedBody(event, schema.parse);
setResponseStatus(event, 201);
return created;
```

### Возможности

- **`getQuery`/`readBody`/`getHeader`/`setCookie`** — H3-хелперы.
- **`useRuntimeConfig(event)`** — секреты и публичные настройки (`runtimeConfig.apiSecret`).
- **Storage** (`useStorage()`) — файловые/Redis драйверы.
- **Кэш** — `routeRules` (`swr`, `cachedEventHandler`), заголовки.
- **Cron/задачи** — Nitro tasks (`nitro.tasks`), расписание — на уровне платформы деплоя.
- **Деплой** — Node-server, serverless (Vercel/Firebase), Workers, Deno, Bun; один и тот же код.

### BFF-паттерн

```
Браузер → /api/* (Nitro) → внешний API (секрет на сервере)
```

- Токены/секреты не попадают в клиентский бандл.
- Cookie ставится на своём домене (`HttpOnly`), а не на чужом API.
- Нормализация данных, агрегация нескольких источников.

### В Nuxt

- Вызов из `useFetch("/api/users")` — на сервере идёт **внутрь** Nitro (без сети), на клиенте — обычный HTTP.
- Server routes **не** попадают в клиентский бандл.
- `server/` не импортируется в `app/` (иначе утечка).

### Ограничения

- Serverless — нет долгих процессов/локального FS (кроме tmp).
- Холодные старты.
- Не «полный» бэкенд, а BFF; тяжёлую бизнес-логику с транзакциями и воркерами выносят.

### Отличие от `server/middleware`

- `server/api` — обработчики эндпоинтов.
- `server/middleware` — на каждый запрос (авторизация, логи).
- `app/middleware` — **клиентские/SSR** навигации (не серверный запрос).

> [!tip] Как отвечать
> «Nitro — сервер Nuxt: server routes по файлам (`server/api/*.get.ts`), H3-хелперы, runtime config, кэш, деплой куда угодно. Это BFF: проксируем API и прячем токены; в отличие от `app/middleware`, это серверные обработчики».

## Связанные вопросы

- [[30 Где выполняется код в Nuxt]]
- [[31 Данные в Nuxt и SSR]]
- [[26 CSR SSR и SSG]]

## Ресурсы

- Nuxt server: https://nuxt.com/docs/guide/directory-structure/server
- Nitro: https://nitro.build/
