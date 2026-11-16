---
type: careers/interview
section: Vue
topic: Nuxt
status: ready
created: 2026-11-16
tags: [interview, nuxt, data-fetching]
---

# Данные в Nuxt и SSR

## Краткий ответ (30 секунд)

**`$fetch`** — низкоуровневый fetch (на основе ofetch): SSR-запрос к API на сервере/в Nitro-роутах, actions; **на сервере не передаёт payload и не делает dedup между компонентами**. **`useAsyncData`** — композабл для данных, которые нужно **отдать с сервера в клиент** через payload: сам запускает на сервере, сериализует, не повторяет на клиенте, даёт `data/pending/error/status/refresh`. **`useFetch`** — обёртка над `useAsyncData` + `$fetch(url)`: «загрузить URL», с автоматическим ключом/реактивными параметрами. Правило: `useFetch` для запросов компонентов/страниц, `useAsyncData` — когда нужен кастомный загрузчик/`$fetch` с логикой, `$fetch` — в серверных обработчиках/действиях.

## Подробно

### `$fetch`

```ts
const user = await $fetch("/api/user/1");
```

- Тонкая обёртка (ofetch): JSON, ошибки HTTP, interceptors.
- **Не** делает SSR-кэш/payload — в `setup` на сервере и клиенте выполнится дважды (дублирующий запрос).
- Правильное место: `server/api/*.ts`, `actions`, `onSubmit`.
- **Не забывайте**: `$fetch("/api/...")` на сервере идёт «внутрь» Nitro без сети (что хорошо), но абсолютный URL к внешнему API — реальный HTTP.

### `useAsyncData`

```ts
const { data, pending, error, refresh } = await useAsyncData("user", () =>
  $fetch("/api/user"),
);
```

- **Ключ** — идентичность данных (должен включать параметры, например `user-${id}`): общий реф, дедупликация.
- **SSR**: выполняется на сервере; результат в `payload`, клиент не повторяет (при гидратации).
- `data` — `Ref<T | undefined>` (в Nuxt 4; ранее `null`), **`shallowRef` по умолчанию** (не глубокая реактивность → `deep: true` при необходимости).
- `refresh`/`execute`/`clear`, `status`, `pending`.
- Опции: `watch` (реактивные зависимости для рефетча), `transform`, `pick`, `getCachedData`, `server: false` (только клиент).

### `useFetch`

```ts
const { data: product } = await useFetch(`/api/products/${id}`, {
  key: `product-${id}`,
  watch: [() => id],
});
```

- То же + `$fetch(url, options)` под капотом.
- Автоматический ключ по URL (осторожно с динамическими URL — добавляйте параметры явно в ключ).
- Реактивный `query`/`params` — список параметров, влияющих на ключ.

### Nuxt 4: отличия

- **Одинаковые ключи → общие refs** между компонентами (согласованность данных).
- **Реактивные ключи** (computed/ref/getter) — при смене ключа новый запрос, старые данные чистятся.
- **Очистка при unmount** последнего потребителя.
- **`data` по умолчанию `undefined`, `shallowRef`**, при рефетче `data` очищается (dashboard: показать loading, не старые данные) — отличается от «сохраняем старые данные» в раннем Nuxt 3. Проверьте `getCachedData`/`deep` при миграции.

### Когда что

| Задача | Инструмент |
|---|---|
| Данные для страницы/компонента с SSR | `useFetch` |
| Свой загрузчик (агрегация, БД, transform) | `useAsyncData` + `$fetch` |
| Серверный роут/Nitro handler | `$fetch` / прямой драйвер |
| Мутация (POST/сохранить) | `$fetch` в action (не `useFetch`!) |
| Бесконечный список | `useAsyncData` с ручным `loadMore`/`refresh` |
| Polling | `useAsyncData` с `watch` или `setInterval` |

### Ошибки

```ts
if (error.value) {
  // 404 — показать not found; иначе — throw createError
}
```

- `useFetch` не бросает при HTTP-ошибке автоматически; `$fetch` бросает `FetchError`.
- `createError({ statusCode: 404, statusMessage: "..." })` — SSR-ответ.
- `error.value.statusCode`/`statusMessage`.

### `useLazyAsyncData` / `useLazyFetch`

- Не блокируют навигацию (нет `await` в setup) — Suspense не ждёт.
- Для второстепенных блоков.

> [!warning] Подвох
> «Почему запрос ушёл дважды?» — `$fetch` в `setup` (или `onMounted`) вместо `useFetch`/`useAsyncData`, либо разные ключи.

## Связанные вопросы

- [[30 Где выполняется код в Nuxt]]
- [[28 SSG и prerender]]
- [[29 Гидратация и hydration mismatch]]

## Ресурсы

- Nuxt data fetching: https://nuxt.com/docs/getting-started/data-fetching
