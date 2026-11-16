---
type: careers/interview
section: Vue
topic: Встроенные компоненты
status: ready
created: 2026-11-16
tags: [interview, vue, teleport, keepalive, transition]
---

# Встроенные компоненты Vue

## Краткий ответ (30 секунд)

- **`<Teleport>`** — рендерит содержимое в **другую часть DOM** (обычно `body`), логика/пропсы остаются в вашем компоненте. Для модалок, тостов, дропдаунов — обойти `overflow: hidden`/`z-index`.
- **`<KeepAlive>`** — **кэширует экземпляры** компонентов при переключении (сохраняет state, не вызывает `unmounted`), `include`/`exclude`/`max`, с хуками `onActivated`/`onDeactivated`.
- **`<Transition>`** — анимации появления/исчезновения одного элемента (`v-if`, `v-show`, `:is`, `key`), CSS-классы `*-enter-active`, `*-leave-active`; **`<TransitionGroup>`** — для списков (`v-for`).
- **`<Suspense>`** — ждёт асинхронные зависимости (async `setup`/асинхронные компоненты), fallback до готовности.
- **`<component :is>`** — динамический компонент; **`<NuxtPage>`/`<RouterView>`** — из экосистемы.

## Подробно

### Teleport

```vue
<Teleport to="body">
  <div v-if="open" class="modal">…</div>
</Teleport>

<!-- к элементу по селектору или ref -->
<Teleport to="#modals">
```

- **Контекст остаётся**: props, provide/inject, события — от исходного компонента.
- `disabled` — отключить телепорт (SSR-friendly).
- Стили: scoped-стили на телепортированные элементы **работают** (data-атрибут остаётся). Но CSS-каскад другого родителя может повлиять — учитывать.
- `Teleport` при SSR: Nuxt рендерит в target, нужен target в HTML.

### KeepAlive

```vue
<KeepAlive :include="['Inbox']" :max="10">
  <component :is="currentTab" />
</KeepAlive>
```

- Не размонтирует — DOM/state остаются.
- `include`/`exclude` — по **имени** компонента (`name` в Options API / `defineOptions({ name })`).
- `max` — LRU-вытеснение.
- `onActivated` — вход, `onDeactivated` — выход; **не** `mounted`/`unmounted`.
- Внутри `deactivated` эффекты (таймеры, подписки) **продолжают** работать — выключайте их (`deactivated`) или используйте `onActivated` для запуска.
- Не работает с Vapor-only-режимом в некоторых границах (см. Vapor interop).

### Transition

```vue
<Transition name="fade" mode="out-in">
  <p :key="step" v-if="currentStep">{{ step }}</p>
</Transition>
```

```css
.fade-enter-active, .fade-leave-active { transition: opacity 0.2s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
```

- Классы: `v-enter-from` → `v-enter-active` → `v-enter-to`, аналогично `leave`.
- `mode="out-in"|"in-out"` — порядок; по умолчанию одновременно.
- `appear` — анимация при первом рендере.
- `<TransitionGroup>` — перемещение `move-class`, ключи обязательны, удобно для списков.

### Suspense

```vue
<Suspense>
  <template #default><AsyncUser /></template>
  <template #fallback><Spinner /></template>
</Suspense>
```

- Ждёт `await` в `setup` (включая `useAsyncData` в Nuxt).
- Ошибка — через `onErrorCaptured`/`errorCaptured` (или `onError`-границы Nuxt).
- Suspense — экспериментальный API, но широко используется; в Nuxt 3 особенно.

### `<component :is>`

```vue
<component :is="isLogin ? LoginForm : RegisterForm" :key="mode" />
```

- Замена `v-if`-веток; с `KeepAlive` — сохранение состояния вкладок.

### Таблица «что решает»

| Компонент | Проблема | Где |
|---|---|---|
| Teleport | stacking/overflow для оверлеев | модалки, тосты |
| KeepAlive | потери state при переключении вкладок | табы, мастера |
| Transition | появление/исчезновение | ховеры, роуты |
| TransitionGroup | анимация списка | удаление элементов |
| Suspense | асинхронная загрузка | async setup, data fetching |

> [!warning] Подвох
> «KeepAlive = `mounted` при возврате?» — нет: вернувшись, компонент переходит в `activated`, `mounted` не вызывается повторно. И таймер, запущенный в `mounted`, продолжит идти в скрытом кэше, если не остановить в `deactivated`.

## Связанные вопросы

- [[03 Хуки жизненного цикла]]
- [[06 Слоты]]
- [[08 Асинхронные компоненты]]

## Ресурсы

- Vue built-in components: https://vuejs.org/api/built-in-components
