---
type: careers/interview
section: Vue
topic: Nuxt
status: ready
created: 2026-11-16
tags: [interview, nuxt, middleware]
---

# Middleware в Nuxt

## Краткий ответ (30 секунд)

Middleware — функции, которые выполняются **перед навигацией** (перед рендером страницы) и могут редиректить, отменять навигацию или что-то подготовить. Виды: **анонимный** (в `definePageMeta` у страницы), **именованный** (`middleware/auth.ts`, подключается в `definePageMeta({ middleware: "auth" })`) и **глобальный** (`middleware/*.global.ts`). Работает и на сервере, и на клиенте; для SSR-защиты важно проверять авторизацию по cookie (`useCookie`), а не по `localStorage`. Для API/бэкенда — **серверный** middleware (`server/middleware`), это другой механизм (Nitro).

## Подробно

### Анонимный

```vue
<!-- pages/admin.vue -->
<script setup lang="ts">
definePageMeta({
  middleware: [
    (to) => {
      if (!useAuth().isLoggedIn) return navigateTo("/login");
    },
  ],
});
</script>
```

### Именованный

```ts
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to) => {
  const token = useCookie("token");
  if (!token.value) {
    return navigateTo({ path: "/login", query: { redirect: to.fullPath } });
  }
});
```

```vue
<script setup lang="ts">
definePageMeta({ middleware: "auth" });
</script>
```

### Глобальный

```
app/middleware/analytics.global.ts
```

Запускается на каждой навигации (в т.ч. на сервере при первой загрузке).

### Что можно вернуть

- `undefined`/`true` — продолжать.
- `navigateTo(...)`/`abortNavigation()` — редирект/отмена.
- `showError(...)`/`createError` — ошибка.

### Порядок

- Глобальные → именованные (из `definePageMeta`) → анонимные.
- Асинхронные — **должны** быть `await` внутри middleware (иначе не отработает).
- `to`/`from` — роуты; `import.meta.client`/`import.meta.server` для условий.

### Middleware и SSR

- На первой загрузке — **сервер**, затем клиент при клиентских переходах.
- Для защиты страницы от неавторизованного пользователя нужны **cookie** (доступны на сервере) или серверный рендер с проверкой.
- `localStorage` в middleware на сервере = ошибка; защита будет «дырявой» на первом рендере.
- Для «только клиент» — проверка `if (import.meta.server) return;`.

### Layouts vs middleware

- **Layout** — визуальная оболочка (сайдбар/футер).
- **Middleware** — логика допуска/подготовки данных. Не путать.

### `server/middleware`

```ts
// server/middleware/auth.ts
export default defineEventHandler((event) => {
  event.context.user = verify(event.node.req.headers.authorization);
});
```

- Это **Nitro middleware**: на каждый API-запрос, на сервере.
- Для аутентификации API, логирования, CORS-заголовков.

### Подвох

- Middleware выполняется при **навигации** (смена `path`/`params`); изменение только `query`/`hash` — не всегда новая навигация, поэтому логику на query лучше держать в компоненте.
- Редирект из middleware в SSR возвращает HTTP 302 — это нормально для ботов.
- Не делать в middleware тяжёлые `useAsyncData` — для этого есть композаблы страницы.

> [!tip] Как отвечать
> «Middleware — гейт перед навигацией: анонимный/именованный/глобальный. В SSR проверяю cookie, а не localStorage, иначе первый рендер утечёт контент. Серверная проверка API — `server/middleware`».

## Связанные вопросы

- [[34 Авторизация в SSR]]
- [[26 CSR SSR и SSG]]
- [[30 Где выполняется код в Nuxt]]

## Ресурсы

- Nuxt route middleware: https://nuxt.com/docs/guide/directory-structure/middleware
