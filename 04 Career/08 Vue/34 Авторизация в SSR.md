---
type: careers/interview
section: Vue
topic: Nuxt
status: ready
created: 2026-11-16
tags: [interview, nuxt, auth, ssr, cookies]
---

# Авторизация в SSR

## Краткий ответ (30 секунд)

В SSR **`localStorage` не существует** на сервере, и сервер не может прочитать его при первом запросе. Поэтому источником авторизации должны быть **cookie** (`HttpOnly`, `Secure`, `SameSite`) — их браузер отправляет с запросом, сервер читает и делает SSR с данными пользователя. Схема в Nuxt: логин → сервер (`server/api/login`) ставит refresh cookie → фронт держит access в памяти/`useState`, SSR-запросы идут через `useFetch` с пробросом cookie (или BFF), страница рендерится авторизованной. `middleware` проверяет cookie до рендера. Профиль — `/api/me` на сервере или `useState`. `localStorage` — максимум «подсказка UI», не источник правды.

## Подробно

### Почему не `localStorage`

1. **Сервер не видит** — первый HTML будет «гостем» (мигание, редирект, SEO/OG неверны).
2. **XSS** читает `localStorage` — долгоживущий токен украден.
3. **Нет автоматической отправки** — каждый запрос нужно вручную заполнять заголовком (в SSR — вообще отдельный канал).
4. **Можно подделать** профиль/роли на клиенте — доступ решается на сервере, не в браузере.

### Схема Nuxt

```
POST /api/login      (server route) → Set-Cookie refresh; return user + access (в памяти)
middleware/auth.ts   → useCookie("refresh"/"token") → navigateTo("/login") если нет
useFetch("/api/me")  → на SSR пробрасывает cookie (same-origin автоматически)
```

- **`useCookie("token")`** — доступно и серверу, и клиенту; `useCookie` умеет `httpOnly`, `secure`, `sameSite`, `maxAge`.
- **`useRequestHeaders(["cookie"])`** во внутренних вызовах к внешнему API.
- **`useRuntimeConfig`** — секретный API-ключ только на сервере.

### Middleware

```ts
// middleware/auth.ts
export default defineNuxtRouteMiddleware(async (to) => {
  const token = useCookie("sb-access-token");
  if (!token.value) {
    return navigateTo(`/login?redirect=${encodeURIComponent(to.fullPath)}`);
  }
});
```

- Для SSR-защиты — cookie, не `localStorage`.
- Для клиентских переходов — тот же middleware, но источник уже в памяти/куке.
- «Только клиентские» проверки делают `if (import.meta.client)`, но это не защита — просто UX.

### Access/refresh

- **Access** — короткий, в **памяти** (Pinia/`useState`), в заголовке к серверным роутам.
- **Refresh** — `HttpOnly` cookie; `POST /api/refresh` ставит новый access и (rotation) refresh.
- При SSR — cookie уходит автоматически в same-origin Nitro-роуты; к внешнему API — явно из `useRequestHeaders`.
- **CSRF**: `SameSite=Lax` + проверка `Origin` на серверных роутах; токен в cookie `SameSite=None` — с `Secure`.

### Сессия после гидратации

- На сервере: кука → `/api/me` → `useState("user", ...)` → HTML с профилем.
- На клиенте: `useState` **уже** из payload (не повторяем запрос); протухший access → refresh-интерцептор в `ofetch`.

Интерцептор:

```ts
// plugins/api.ts
export default defineNuxtPlugin(() => {
  const api = $fetch.create({
    onResponseError: async ({ response, request, options }) => {
      if (response.status === 401) {
        await $fetch("/api/refresh", { method: "POST" });
        return $fetch(request, options); // повтор
      }
    },
  });
  return { provide: { api } };
});
```

### Платформы (Supabase/Auth0/Firebase)

- Используйте SSR-модули (`@nuxtjs/supabase`, `sidebase`), которые кладут сессию в cookie.
- Иначе `useState`/localStorage — и SSR окажется «гостевым».

### Ловушки

- `localStorage` в plugin на сервере → падение.
- Токен в `query`/URL — утечка в логи/Referer.
- Один глобальный `let user` — утечка между запросами.
- Проверка ролей только на клиенте (кнопка скрыта, API открыт).
- Cookie на `Domain=.example.com` — доступна всем поддоменам.

> [!warning] Что ожидают
> «localStorage нельзя как единственный источник в SSR — сервер его не видит, и это небезопасно. Cookie HttpOnly + серверная валидация + BFF — правильный ответ».

## Связанные вопросы

- [[25 Хранение auth состояния]]
- [[32 Middleware в Nuxt]]
- [[33 Nitro и server routes]]

## Ресурсы

- Nuxt useCookie: https://nuxt.com/docs/api/composables/use-cookie
- Sidebase auth: https://auth.sidebase.io/
