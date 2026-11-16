---
type: careers/interview
section: React
topic: Оптимизация
status: ready
created: 2026-11-16
tags: [interview, react, lazy, suspense]
---

# React.lazy и Suspense

## Краткий ответ (30 секунд)

**`React.lazy(() => import(...))`** — ленивая загрузка компонента: модуль подгружается, когда компонент впервые рендерится. **`<Suspense fallback={...}>`** — показывает `fallback`, пока ленивый компонент (или данные через `use`, или `useTransition`) не готов. Это основа **code-splitting**: уменьшает начальный JS-бандл и ускоряет LCP/INP. Правила: `Suspense` должен быть **вокруг** ленивого компонента, `lazy` — на верхнем уровне модуля (не внутри `render`), fallback — без «прыжков» (иначе CLS), для маршрутов — граница на маршрут. Ошибки — через Error Boundary рядом.

## Подробно

### Базовый пример

```jsx
const Chart = React.lazy(() => import("./Chart"));

function Dashboard() {
  return (
    <React.Suspense fallback={<ChartSkeleton />}>
      <Chart data={data} />
    </React.Suspense>
  );
}
```

`import("./Chart")` создаёт отдельный чанк; при монтировании он грузится; до загрузки — `fallback`.

### Роут-сплиттинг (самый ценный)

```jsx
const Settings = lazy(() => import("./pages/Settings"));

<Routes>
  <Route path="/settings" element={
    <Suspense fallback={<PageSkeleton />}>
      <Settings />
    </Suspense>
  } />
</Routes>
```

Каждый маршрут — отдельный чанк. Начальная загрузка содержит только «оболочку» + текущую страницу.

### Suspense умеет больше

- **Данные**: `use(promise)` (React 19) внутри `Suspense`; React suspend-ит рендер.
- **`useTransition`**: `startTransition` + `Suspense` = «плавный переход» без замены старого UI.
- **`useDeferredValue`**: откладывает тяжёлый рендер, показывая старый.
- **`Activity`** (19.2): скрывает поддерево, сохраняя состояние; может рендерить в фоне.
- **SSR-стриминг** (`renderToPipeableStream`): Suspense-границы отправляются по мере готовности.

### Правила и подвохи

```jsx
// ❌ создаётся новый lazy на каждом рендере — компонент будет размонтироваться
function App() {
  const Comp = lazy(() => import("./Comp")); // NOPE
}
```

- **`lazy` — только на верхнем уровне модуля.**
- **`Suspense` должен охватывать** ленивый компонент (в том числе во вложенных местах — если компонент ленивый, но рендерится в `<ul>`, граница должна быть выше).
- **`fallback`** должен иметь тот же размер, чтобы не прыгал layout (CLS).
- **Сбой загрузки чанка** (плохая сеть, старый хеш после деплоя) — `lazy` бросает ошибку; рядом нужен **Error Boundary** с retry/reload.
- Лениво грузить **много мелких** компонентов = много запросов; лениво грузить то, что нужно сразу (above-the-fold) — ухудшит LCP.
- `React.lazy` не поддерживает именованный экспорт напрямую:

```jsx
const Chart = lazy(() => import("./chart").then((m) => ({ default: m.Chart })));
```

### Что не стоит лениво грузить

- Компонент первого экрана.
- «Хром» (шапка, меню) — нужен сразу.
- Компонент, который отрисовывается через 50 мс после загрузки — ленивость только добавит запрос.

### Метрики

- Уменьшение **initial bundle** → лучше LCP/TBT.
- Prefetch на hover/`link rel="prefetch"` — по клику уже готово.
- Vite/webpack: `import()` и `manualChunks` для осмысленных границ.

> [!tip] Как отвечать
> «`lazy` — динамический import, `Suspense` — fallback на время загрузки. Классика — code-splitting по маршрутам. Лениво только то, что не нужно на первом экране; fallback фиксированного размера; ошибки загрузки — через Error Boundary».

## Связанные вопросы

- [[Как оптимизировать загрузку тяжёлых компонентов]]
- [[Что нового в React 18 и 19]]
- [[Избежание лишних ререндеров]]

## Ресурсы

- react.dev lazy: https://react.dev/reference/react/lazy
- react.dev Suspense: https://react.dev/reference/react/Suspense
