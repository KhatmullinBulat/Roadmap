---
type: careers/interview
section: Vue
topic: Nuxt
status: ready
created: 2026-11-16
tags: [interview, nuxt, hydration, ssr]
---

# Гидратация и hydration mismatch

## Краткий ответ (30 секунд)

**Гидратация** — клиент берёт серверный HTML и «оживляет» его: Vue монтирует компоненты, привязывает обработчики, восстанавливает состояние, **повторяя тот же рендер** с теми же данными (из payload). **Mismatch** возникает, когда результат на сервере и клиенте различается: сервер отрисовал одно, клиент — другое. Причины: `Date.now()`/`Math.random()`, `localStorage`/`window` в `setup`, разный порядок/набор данных, неверные ключи, `v-if` по клиентским условиям, расширения браузера/нестабильные id, `v-for` без `key`. Nuxt предупреждает в консоли (dev), а в prod возможна «перерисовка» и исчезновение DOM.

## Подробно

### Что происходит

1. Сервер: `setup` → HTML + `__NUXT__` payload.
2. Клиент: скачивает HTML (виден контент), затем JS.
3. Hydration: Vue создаёт компоненты, **проверяет** соответствие серверному DOM, привязывает события.
4. После — обычное клиентское поведение.

### Типовые причины

| Причина | Пример | Лечение |
|---|---|---|
| Время/случайность | `new Date().toLocaleTimeString()` | `NuxtTime`, `ClientOnly`, стабильный формат |
| `window`/`localStorage` в setup | `localStorage.getItem("theme")` | `useState`, `useCookie`, `onMounted` |
| Разные данные | запрос на сервере без payload | `useAsyncData`/`useFetch` (payload доедет) |
| `v-if` по клиенту | `navigator.onLine` в шаблоне | `isClient` + `onMounted`, `ClientOnly` |
| `v-for` без `key` | список из Map | стабильные `id` |
| Невалидный HTML | `<p><div>` | исправить вложенность (браузер перестроит) |
| Расширения | вставка плагинами | не тестировать с расширениями, проверять инкогнито |
| Глобальный state | `Math.random()` в сторе | инициализация в `onMounted`/сервер |
| Локаль/таймзона | `toLocaleDateString()` без `timeZone` | фиксировать `timeZone`/`locale` |

### Пример «плохо / хорошо»

```vue
<!-- ❌ сервер и клиент дадут разное -->
<p>{{ new Date().toString() }}</p>

<!-- ✅ -->
<NuxtTime :datetime="date" locale="ru-RU" />
```

```vue
<!-- ❌ localStorage в setup -->
<script setup>
const token = localStorage.getItem("token"); // на сервере ошибка
</script>

<!-- ✅ -->
<script setup>
const token = useCookie("token"); // доступно и серверу, и клиенту
</script>
```

```vue
<template>
  <ClientOnly>
    <MapWidget /> <!-- только клиент -->
  </ClientOnly>
</template>
```

### `ClientOnly` и `isClient`

- `<ClientOnly>` — рендерит только на клиенте (фолбэк на сервере).
- `import.meta.client` / `import.meta.server` — условия.
- `onMounted` — только клиент.
- Не злоупотреблять: каждый `ClientOnly` — минус SSR-пользы.

### `useState` в Nuxt

- `useState` — SSR-safe общий стейт (payload), в отличие от глобальной `ref` в модуле (утечка между запросами на сервере!).

### Payload

- `useAsyncData`/`useFetch` **переносят данные** с сервера в клиент через payload — клиент не делает повторный запрос при гидратации (и не должен).
- Если делаете `fetch` в `setup` напрямую — сервер и клиент сделают два разных запроса → mismatch. Это частая ошибка.

### Дети Vapor/Teleport

- `Teleport` при SSR: target должен быть в HTML сервера.
- `Suspense`-фолбэки — гидратируются корректно в Nuxt.

> [!warning] Подвох
> «В dev предупреждение, в prod всё выглядит нормально» — это не значит, что mismatch нет: Vue может «пересоздать» DOM, теряя обработчики/фокус/первое состояние; CLS и баги появляются позже. Dev-warning не игнорировать.

## Связанные вопросы

- [[30 Где выполняется код в Nuxt]]
- [[26 CSR SSR и SSG]]
- [[28 SSG и prerender]]

## Ресурсы

- Nuxt SSR/hydration: https://nuxt.com/docs/guide/concepts/rendering#universal-rendering
- Vue hydration: https://vuejs.org/guide/scaling-up/ssr#hydration-mismatch
