---
type: careers/interview
section: React
topic: Хуки
status: ready
created: 2026-11-16
tags: [interview, react, hooks]
---

# Хуки React — обзор

## Краткий ответ (30 секунд)

Чаще всего используются: `useState` (локальное состояние), `useEffect` (синхронизация с внешним миром), `useRef` (значение без ререндера/ссылка на DOM), `useContext` (чтение контекста), `useMemo`/`useCallback` (мемоизация значения/функции), `useReducer` (сложное состояние), `useTransition`/`useDeferredValue` (приоритеты), `useId` (стабильные id для SSR), `useLayoutEffect` (измерения до paint), и в React 19 — `use`, `useActionState`, `useOptimistic`, `useEffectEvent` (19.2). Плюс кастомные хуки как способ переиспользования логики.

## Подробно

### Базовые

| Хук | Для чего | Ключевое |
|---|---|---|
| `useState` | локальное состояние | ленивый инициализатор `useState(() => ...)` |
| `useEffect` | побочные эффекты после paint | cleanup; deps-массив |
| `useRef` | изменяемая «коробка» или DOM | не вызывает ререндер |
| `useContext` | чтение контекста | подписывает на изменения контекста |
| `useReducer` | состояние с переходами | удобно для сложной логики/actions |
| `useMemo` | кэш вычисления | дорогое вычисление |
| `useCallback` | кэш функции | для `memo`-детей/зависимостей эффектов |
| `useLayoutEffect` | эффект **до** paint | измерения, синхронный DOM |
| `useId` | стабильный id | SSR-согласованность |
| `useImperativeHandle` | «публичное» API через `ref` | вместе с `forwardRef` |

```jsx
const [user, setUser] = useState(null);
const inputRef = useRef(null);
const theme = useContext(ThemeContext);
const sorted = useMemo(() => items.toSorted(byDate), [items]);
const onClick = useCallback(() => setOpen(true), []);
```

### Приоритеты и отзывчивость (React 18+)

- `useTransition()` — `startTransition(() => setState(...))`: обновление как низкоприоритетное; `isPending`.
- `useDeferredValue(value)` — отложенная версия значения (для тяжёлого списка).
- **`useSyncExternalStore`** — правильная подписка на внешний стор (Redux/Zustand/`window`).

### React 19

- **`use(promise | context)`** — чтение ресурса в рендере (с Suspense); можно вызывать условно.
- **`useActionState(action, initial)`** — состояние формы + pending.
- **`useFormStatus()`** — статус родительской формы (внутри `<form action>`).
- **`useOptimistic(state, update)`** — оптимистичные обновления.
- **`useEffectEvent`** (19.2) — «свежий» пропс в эффекте без перезапуска эффекта.

### Правила хуков

1. Вызывать **только на верхнем уровне** компонента/кастомного хука — не в условиях, циклах, вложенных функциях.
2. Только из компонента React или другого хука.
3. Порядок вызовов стабилен: состояние хранится по порядку, а не по имени.

Почему: хуки — массив в Fiber. Условие изменит индексы → «Rendered fewer hooks than expected».

### Кастомные хуки

```js
function useDebouncedValue(value, delay = 300) {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}
```

Кастомный хук **не** делит состояние между компонентами: у каждого вызова свой.

### Чего спрашивают дальше

- Почему `useEffect` — это не жизненный цикл (он про синхронизацию, а не про «после рендера»).
- Почему `useState` не синхронизируется сразу (batched, async).
- Когда `useMemo`/`useCallback` **вредят**: дороже создание зависимостей; без `memo` детей не нужны.
- `useRef` vs `useState`: изменение ref не ререндерит.

> [!tip] Как отвечать
> «На практике 80% — `useState`, `useEffect`, `useRef`, `useContext`. Мемоизация — только там, где есть измеренная проблема или `memo`-ребёнок. В React 19 добавились `use`, actions и `useOptimistic`».

## Связанные вопросы

- [[useEffect и useLayoutEffect]]
- [[useMemo и useCallback]]
- [[Что нового в React 18 и 19]]

## Ресурсы

- react.dev Hooks: https://react.dev/reference/react
