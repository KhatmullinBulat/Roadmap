---
type: careers/interview
section: React
topic: Внутреннее устройство
status: ready
created: 2026-11-16
tags: [interview, react, fiber, internals]
---

# Что такое Fiber

## Краткий ответ (30 секунд)

**Fiber** — это внутренняя архитектура реконсиляции React (с версии 16). Каждый узел дерева — объект (**Fiber**) с типом, пропсами, состоянием хуков, ссылкой на DOM-узел (`stateNode`), детьми и соседями, а также **приоритетами** и ссылкой на «альтернативное» дерево (alternate). Главное отличие от старого стека вызовов — рендер можно **приостанавливать, прерывать и разбивать на единицы работы** (concurrent rendering), что позволяет не блокировать браузер и обрабатывать важные обновления первыми. Fiber даёт React возможность двух фаз: **render** (можно прерывать) и **commit** (нельзя).

## Подробно

### Почему «Fiber»

До React 16 реконсиляция — рекурсивный обход обычным стеком вызовов. Если дерево большое, JS занимает поток до конца — UI замирает. Нельзя: прервать, приоритизировать, продолжить.

Fiber превращает рекурсию в **связный список-дерево** с явными указателями. Обход становится итеративным: можно остановиться на любом узле и вернуться позже.

### Структура узла (упрощённо)

```ts
type Fiber = {
  tag: WorkTag;            // FunctionComponent, ClassComponent, HostComponent...
  type: any;               // компонент/тег
  key: string | null;
  elementType: any;
  stateNode: any;          // DOM-узел | экземпляр класса | null
  return: Fiber | null;    // родитель
  child: Fiber | null;     // первый ребёнок
  sibling: Fiber | null;   // следующий брат
  index: number;
  pendingProps: any;
  memoizedProps: any;
  memoizedState: any;      // для хуков — связный список
  updateQueue: any;
  lanes: Lanes;            // приоритеты
  alternate: Fiber | null; // старое дерево (double buffering)
  flags: Flags;            // эффекты commit
  deletions: Fiber[] | null;
};
```

### Два дерева

- **current** — то, что отрисовано.
- **workInProgress** — то, что строится.
- Каждый узел ссылается на двойника через `alternate` — это **double buffering**. Commit переключает `root.current`.

### Фазы

| Фаза | Что | Прерывание |
|---|---|---|
| **Render** | вызов компонентов, реконсиляция, сравнение, планирование эффектов | ✅ можно прервать |
| **Commit** | мутация DOM, layout-эффекты, пассивные эффекты | ❌ синхронно |

- **`beginWork`** — «спускаемся» в потомков, вычисляем новые пропсы/state, планируем обновления.
- **`completeWork`** — «поднимаемся», собираем DOM-изменения.
- **`commitRoot`** — применяет мутации, вызывает `useLayoutEffect`, затем paint, затем `useEffect`.

### Приоритеты (Lanes)

React 18 заменил старую шкалу `expirationTime` битовой маской **lanes**. Разные обновления получают разные полосы:
- `SyncLane` — дискретный ввод (клик);
- `InputContinuousLane` — drag/scroll;
- `DefaultLane` — обычный setState;
- `TransitionLane` — `startTransition`;
- `IdleLane` — фоновое.

Scheduler отдаёт работу браузеру, если бюджет кадра исчерпан (≈5 мс на unit работы), и продолжает в idle-время. Это и есть **time slicing**.

### Хуки и Fiber

Хуки хранятся как **связный список** в `fiber.memoizedState`. Поэтому:
- порядок вызовов должен быть стабильным;
- условие меняет длину списка → «Rendered fewer/more hooks».

```js
// useReducer/useState создают hook-объект { memoizedState, queue, next }
```

### Suspense, Error Boundary, контекст — эффекты Fiber

- **Suspense**: fiber с `tag: SuspenseComponent`; при промис-исключении он «перехватывает» и показывает fallback.
- **Error Boundary**: классовый fiber ловит ошибку в `componentDidCatch`.
- **Context**: провайдер — fiber; потребители — `readContext` при рендере.

### Что отвечать на «зачем вам знать Fiber»

Практические следствия:
1. Почему `startTransition` помогает — обновление получает низкий приоритет.
2. Почему `useLayoutEffect` синхронный — commit-фаза блокирующая.
3. Почему порядок хуков важен — связный список.
4. Почему key критичен — реконсиляция списков.
5. Почему StrictMode дважды запускает эффекты — проверка «чистоты» render-фазы.

> [!tip] Элегантный ответ
> «Fiber — это виртуальное дерево с явными ссылками и приоритетами. Благодаря ему реконсиляция стала прерываемой: React разбивает работу на юниты и уступает браузеру. Поэтому в React 18 можно ставить ввод выше фоновых вычислений».

## Связанные вопросы

- [[Что такое Virtual DOM]]
- [[Что нового в React 18 и 19]]
- [[Избежание лишних ререндеров]]

## Ресурсы

- React blog Fiber: https://legacy.reactjs.org/blog/2017/09/26/react-v16.0.html
- React 18 «Concurrent Rendering»: https://react.dev/blog/2022/03/29/react-v18
