---
type: careers/interview
section: React
topic: Эффекты
status: ready
created: 2026-11-16
tags: [interview, react, hooks, effects]
---

# useEffect vs useLayoutEffect

## Краткий ответ (30 секунд)

Оба выполняют побочный эффект после отрисовки, но в разное время. **`useLayoutEffect`** срабатывает **синхронно после мутации DOM, но до paint** — если внутри меняете DOM/читаете геометрию, пользователь не увидит промежуточное состояние. **`useEffect`** — **после paint**, асинхронно: UI уже показан, эффект не блокирует кадр. Правило: `useLayoutEffect` только для измерений и синхронной коррекции layout; во всём остальном — `useEffect`. SSR: `useLayoutEffect` выдаёт предупреждение (сервер не может его выполнить), используйте `useEffect` или `useIsomorphicLayoutEffect`.

## Подробно

### Порядок в жизни компонента

```
render → commit (DOM обновлён) → useLayoutEffect → paint → useEffect
```

- **`useLayoutEffect`** — вызывается синхронно, **до** того как браузер отрисует. Cleanup — перед следующим layout-эффектом.
- **`useEffect`** — после paint, в микрозадаче/отдельном такте. Поэтому может «мигнуть» промежуточное состояние.

### Когда `useLayoutEffect`

```jsx
// Измерение размера до отрисовки: иначе пользователь увидит неверную позицию
const ref = useRef(null);
const [tooltipLeft, setTooltipLeft] = useState(0);

useLayoutEffect(() => {
  const { width } = ref.current.getBoundingClientRect();
  setTooltipLeft(window.innerWidth - width); // синхронный пересчёт
}, []);
```

Типичные кейсы:
- `getBoundingClientRect`, `offsetWidth`, `scrollHeight`;
- скролл к элементу после рендера;
- подгонка размеров (авторазмер textarea);
- интеграции с не-React библиотеками, которым нужен точный layout.

### Почему не всегда `useLayoutEffect`

- Блокирует paint: долгий синхронный layout-эффект -> видимая задержка кадра (INP/LCP).
- При SSR не выполняется — React выдаёт warning и просит `useEffect`.
- В большинстве случаев «после paint» достаточно: если невидимая работа — зачем блокировать кадр?

### «Мигание» — практический пример

```jsx
// useEffect: пользователь увидит tooltip слева 0 → потом прыжок
// useLayoutEffect: он сразу появится в правильной позиции
```

### Зависимости — та же логика, что у `useEffect`

- `[]` — один раз после mount (+ cleanup при unmount).
- `[a, b]` — при изменении любого.
- без массива — после каждого рендера.
- Функции/объекты в deps меняются каждый рендер — используйте мемоизацию или переносите внутрь эффекта.

### StrictMode и двойной вызов

В dev-режиме React намеренно монтирует/размонтирует/монтирует эффект, чтобы проверить корректность cleanup. Если что-то осталось — это баг эффекта, а не ошибка React.

### Что нового: `useEffectEvent` (19.2)

```jsx
const onConnected = useEffectEvent(() => showToast(theme)); // «свежий» theme
useEffect(() => {
  connect(room, onConnected);
  return () => disconnect();
}, [room]); // theme не в deps
```

Решает «устаревшее замыкание»: `theme` не перезапускает эффект.

> [!tip] Кратко
> «`useLayoutEffect` — до paint, синхронно, для layout-измерений; `useEffect` — после paint, для всего остального. На SSR — только `useEffect`».

## Связанные вопросы

- [[Таймер в компоненте]]
- [[Хуки React — обзор]]
- [[Избежание лишних ререндеров]]

## Ресурсы

- react.dev: https://react.dev/reference/react/useLayoutEffect
- react.dev «You Might Not Need an Effect»: https://react.dev/learn/you-might-not-need-an-effect
