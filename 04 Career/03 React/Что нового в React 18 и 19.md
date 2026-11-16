---
type: careers/interview
section: React
topic: Версии
status: ready
created: 2026-11-16
tags: [interview, react, versions]
---

# Что нового в React 18 и React 19

## Краткий ответ (30 секунд)

**React 18** — concurrent rendering: `createRoot`, автоматический **batching** (в промисах/timeout), `startTransition`/`useTransition`, `useDeferredValue`, `useId`, `useSyncExternalStore`, Suspense на сервере и **стриминговый SSR**. **React 19** (декабрь 2024) — **Actions** (async-функции в `<form action>`, `useActionState`, `useOptimistic`, `useFormStatus`), **`use()`** для чтения промисов/контекста, **`ref` как проп** (без `forwardRef`), **metadata/`<title>`/`<link>`** внутри компонентов, **React Compiler** (opt-in), улучшения Suspense и SSR. **19.2** (окт. 2025) — `Activity`, `useEffectEvent`, `cacheSignal`, Performance Tracks в DevTools.

## Подробно

### React 18

- **`createRoot`** — новый корень; `ReactDOM.render` удалён.
- **Автоматический batching**: `setState` в промисах/`setTimeout`/нативных обработчиках теперь тоже группируется.
- **`startTransition`** и **`useTransition`** — помечать обновления как низкоприоритетные.
- **`useDeferredValue`** — отложить перерисовку значения.
- **`useId`** — стабильные id для SSR.
- **`useSyncExternalStore`** — корректная подписка на внешние сторы (Zustand/Redux).
- **Suspense** на сервере, **`renderToPipeableStream`** — стриминг HTML.
- **Concurrent features** работают только в `createRoot`; в legacy-`render` — нет.
- **Strict Mode** стал активнее: двойной mount эффектов в dev.

```jsx
import { createRoot } from "react-dom/client";
createRoot(document.getElementById("root")).render(<App />);
```

### React 19

**Actions**

```jsx
const [error, submit, isPending] = useActionState(async (prev, formData) => {
  const res = await save(Object.fromEntries(formData));
  return res.error ?? null;
}, null);

<form action={submit}>
  <input name="title" />
  <button disabled={isPending}>Сохранить</button>
</form>
```

- `<form action={asyncFn}>` — функция получает `FormData`, форма сбрасывается.
- **`useActionState`** — результат + pending.
- **`useOptimistic`** — оптимистичное обновление.
- **`useFormStatus`** — статус формы из дочернего компонента.

**`use()`**

```jsx
const comments = use(commentsPromise); // Suspense до готовности
const theme = use(ThemeContext);       // можно условно, в отличие от useContext
```

Внимание: промис должен быть **стабильным** (из библиотеки/кеша), нельзя создавать его в рендере.

**`ref` как проп**

```jsx
function MyInput({ placeholder, ref }) {
  return <input placeholder={placeholder} ref={ref} />;
}
<MyInput ref={inputRef} /> // без forwardRef
```

`forwardRef` устаревает.

**Документ-метаданные**

- `<title>`, `<meta>`, `<link>` можно рендерить прямо в компонентах — React сам поднимет их в `<head>`.
- `<style>` и `<link rel="stylesheet" precedence="...">` с дедупликацией и порядком.

**Прочее**

- Встроенная поддержка `<style>`/`precedence`, `preload`/`preinit`.
- Улучшения SSR/Suspense (пре-прогрев).
- **React Compiler** — babel-плагин, автоматически мемоизирует; stabilish, opt-in.
- Удалены legacy: `defaultProps` для функций, `propTypes` для функций, string refs, `ReactDOM.render`.

### React 19.2 (октябрь 2025)

- **`<Activity mode="visible" | "hidden">`** — скрыть поддерево, сохранив state; эффекты отключаются. Позволяет «прогревать» вкладки.
- **`useEffectEvent`** — «свежие» значения в эффектах без перезапуска.
- **`cacheSignal`** — сигнал отмены для RSC-кэша.
- **Performance Tracks** — в DevTools видны приоритеты React Scheduler и компоненты.
- `prerender()`/`resume()` — Partial Pre-Rendering.

### Что отвечать, если спросят «в чём смысл 18»

> «React 18 — это архитектурный переход на concurrent rendering. Приоритеты: ввод важнее вычислений. Поэтому появились `startTransition`/`useDeferredValue`, `useSyncExternalStore` для сторов и стриминговый SSR».

### Что отвечать про 19

> «React 19 — про UX и меньше boilerplate: Actions и формы без ручных `isPending`, `use()` для ресурсов, `ref` без `forwardRef`, метаданные в компонентах. Плюс Compiler, который снимает боль ручной мемоизации».

## Связанные вопросы

- [[Что такое Fiber]]
- [[Библиотеки для форм RHF и RFF]]
- [[React.lazy и Suspense]]
- [[Хуки React — обзор]]

## Ресурсы

- React 18: https://react.dev/blog/2022/03/29/react-v18
- React 19: https://react.dev/blog/2024/12/05/react-19
- React 19.2: https://react.dev/blog/2025/10/01/react-19-2
