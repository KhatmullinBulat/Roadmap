---
type: careers/interview
section: React
topic: Оптимизация
status: ready
created: 2026-11-16
tags: [interview, react, memo, hooks]
---

# useMemo vs useCallback

## Краткий ответ (30 секунд)

`useMemo(fn, deps)` кэширует **результат** функции; `useCallback(fn, deps)` кэширует **саму функцию**. Второй эквивалентен `useMemo(() => fn, deps)`. `useMemo` нужен для дорогих вычислений и стабильной ссылки на **значение** (объект/массив для `memo`-детей или зависимостей эффектов). `useCallback` — когда важен **идентификатор функции**: для `memo`-ребёнка, для зависимостей эффекта, для подписки/`removeEventListener`. Оба не «гарантия», а совет; без измерений их часто добавляют зря (и React Compiler в 19 делает это автоматически).

## Подробно

### Фундаментальная разница

```jsx
const value = useMemo(() => compute(a, b), [a, b]); // значение
const handler = useCallback((x) => doSomething(x, a), [a]); // функция
```

`useCallback` = `useMemo(() => (x) => ..., [deps])`. Это не две разные механики, а два удобных имени.

### Когда `useMemo` реально полезен

1. **Дорогое вычисление:**

```jsx
const sorted = useMemo(() => rows.toSorted(compare), [rows]);
```

2. **Стабильная ссылка** для `useEffect`/`memo`-ребёнка:

```jsx
const options = useMemo(() => ({ pageSize, sort }), [pageSize, sort]);
fetchList(options); // не перезапускается без причины
```

3. **Сложная конструкция** в зависимостях (массив, объект).

### Когда `useMemo` вреден/бесполезен

- **Примитивы:** `useMemo(() => a + b, [a, b])` — сравнение дешевле, чем кэш.
- **При каждом рендере зависимости новые** (`[items.filter(...)]`) — кэш не срабатывает, дополнительный код.
- `useMemo` — не гарантия: React может «забыть» кэш (например, при размонтировании). Не полагайтесь на него для сайд-эффектов.

### Когда `useCallback`

1. `memo`-ребёнок:

```jsx
const onSelect = useCallback((id) => setSelected(id), []);
<Row item={item} onSelect={onSelect} />
```

Без `useCallback` — новый проп `onSelect` каждый рендер → `memo` бесполезен.

2. Зависимость эффекта:

```jsx
const load = useCallback(() => fetchUser(id), [id]);
useEffect(() => { load(); }, [load]);
```

3. Подписка/`addEventListener`:

```jsx
const handler = useCallback(() => {...}, [deps]);
useEffect(() => {
  window.addEventListener("resize", handler);
  return () => window.removeEventListener("resize", handler);
}, [handler]);
```

4. В библиотеке компонентов (публичный API) — стабильность для потребителей.

### Что не так с мемоизацией «на всякий случай»

- Каждый `useMemo`/`useCallback` — объект в памяти + сравнение зависимостей на каждый рендер.
- Больше кода → хуже читаемость.
- **Часто причина ререндеров не там**: нестабильный `value` контекста, глобальный стор, отсутствие границ компонентов.
- **React Compiler** (React 19, opt-in) автоматически вставляет нужную мемоизацию — ручной код остаётся для случаев, где компилятор не может доказать или где параметр `deps` важен для поведения.

### Антипаттерн

```jsx
// ❌ useCallback с пустыми deps и замыканием на устаревшее значение
const onClick = useCallback(() => {
  setCount(count + 1); // count «заморожен» на первом рендере
}, []);
```

Правильно: `setCount(c => c + 1)` или указать `count` в deps. Линтер `react-hooks/exhaustive-deps` как раз об этом.

### Практическое правило

- Сначала — измерьте (Profiler, `why-did-you-render`).
- Мемоизируйте на **границах тяжёлых поддеревьев**, а не всё подряд.
- Если нужен стабильный `callback` в `memo`-ребёнке, передавайте `id` **через проп**, а не замыкайте в функции — тогда функция не зависит от ничего.

```jsx
// ✅ вместо этого:
<Row id={id} onSelect={onSelect} /> // onSelect = (id) => ...
// где onSelect стабилен: const onSelect = useCallback((id) => setSelected(id), []);
```

> [!tip] Как отвечать
> «`useMemo` кэширует значение, `useCallback` — функцию, и это одно и то же. Полезны для дорогих вычислений и стабильных ссылок в `memo`-детях и эффектах. Зря не оборачиваю: без измерения это лишняя сложность, а с React Compiler — часто вообще не нужно».

## Связанные вопросы

- [[Почему memo не всегда работает]]
- [[Избежание лишних ререндеров]]
- [[Хуки React — обзор]]

## Ресурсы

- react.dev useMemo: https://react.dev/reference/react/useMemo
- react.dev useCallback: https://react.dev/reference/react/useCallback
