---
type: careers/interview
section: Vue
topic: Nuxt
status: ready
created: 2026-11-16
tags: [interview, nuxt, ssr, server]
---

# Где выполняется код в Nuxt

## Краткий ответ (30 секунд)

Код приложения **изоморфный**: один и тот же `setup` выполняется **на сервере** (первый запрос) и **в браузере** (гидратация). На сервере нет `window`, `document`, `localStorage`, `navigator` — обращение к ним даст ошибку (`500` в SSR) или mismatch. В Nuxt есть строгие границы: `import.meta.server`/`import.meta.client`, `server/`-директория (только сервер, Nitro), `shared/` (изоморфно), `app/` (клиент/SSR). Для браузерных API — `onMounted`, `ClientOnly`, `process.client` (старый), плагины `{ mode: "client" }`. SSR-данные — через `useAsyncData`/`useFetch`/`useCookie`, чтобы сервер и клиент видели одинаковое.

## Подробно

### Что есть только на клиенте

- `window`, `document`, `navigator`, `location`.
- `localStorage`, `sessionStorage` (в браузере; на сервере — нет).
- `IntersectionObserver`, `ResizeObserver`, `matchMedia`.
- `requestAnimationFrame`, `WebSocket` (в Node 22+ есть частично, но не как в браузере).

```vue
<script setup>
onMounted(() => {
  window.addEventListener("scroll", handler); // ✅ только клиент
});
</script>
```

### Флаги

| Флаг | Где |
|---|---|
| `import.meta.client` / `import.meta.server` | в коде/Vite |
| `process.client` / `process.server` | устаревшие, но встречаются |
| `<ClientOnly>` | шаблон |
| `{ mode: "client" }` у плагина | только клиент |
| `server/` | сервер (Nitro) |

### Директории Nuxt 4

```
app/        — Vue-код (компоненты, страницы, композаблы)
server/     — только сервер: api/, routes/, middleware/, plugins/
shared/     — изоморфный код без Vue/сервера
public/     — статика
```

Серверный код **не попадает** в клиентский бандл, если он в `server/`. Но `import` из `server/` в `app/` — ошибка/утечка.

### Плагины

```ts
// plugins/analytics.client.ts
export default defineNuxtPlugin(() => {
  // выполнится только в браузере
});
```

- `.client.ts`/`.server.ts` в имени — автоматическая фильтрация.
- Порядок — `dependsOn`/`order` (число).

### SSR-данные

```vue
<script setup>
const { data } = await useFetch("/api/user");   // сервер → payload → клиент (без повторного)
const token = useCookie("token");               // куки доступны на сервере
const theme = useState("theme", () => "light"); // SSR-safe общий стейт
</script>
```

- Обычный `fetch` в `setup` — выполнится и на сервере, и на клиенте (двойной запрос, mismatch).
- `useCookie` читает и пишет cookie; на сервере — из `event.node.req.headers.cookie`.

### `useRequestEvent` / `useRequestHeaders`

```ts
const headers = useRequestHeaders(["cookie"]); // только на сервере (клиент — пусто)
```

- Для проброса заголовков на бэкенд (`$fetch("/api", { headers })`).
- `useRequestURL()` — абсолютный URL (для SSR-редиректов).

### Ошибки, которые чаще всего ловят

1. `localStorage` в `setup` → 500 в SSR.
2. `window` в `computed`/шаблоне → mismatch/ошибка.
3. Прямой `$fetch` в `setup` → двойной запрос.
4. Глобальная переменная в модуле (`let user`) → утечка между пользователями (сервер один процесс на много запросов).
5. `Math.random`/`Date.now` в рендере → mismatch.

> [!warning] Главное
> «Изоморфность» — это не «код работает везде», а «код должен выполняться корректно в двух средах». Сначала спросите: «Нужно ли это на сервере?» Если нет — `onMounted`/`ClientOnly`.

## Связанные вопросы

- [[29 Гидратация и hydration mismatch]]
- [[31 Данные в Nuxt и SSR]]
- [[33 Nitro и server routes]]

## Ресурсы

- Nuxt runtime config: https://nuxt.com/docs/guide/going-further/runtime-config
- Nuxt rendering: https://nuxt.com/docs/guide/concepts/rendering
