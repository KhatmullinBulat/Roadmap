---
type: careers/interview
section: React
topic: Оптимизация
status: ready
created: 2026-11-16
tags: [interview, react, memo, hoc]
---

# HOC и React.memo

## Краткий ответ (30 секунд)

**HOC (Higher-Order Component)** — функция, принимающая компонент и возвращающая новый компонент: `withAuth(Component)`. Это паттерн переиспользования логики (props-инъекции, auth, логирование). **`React.memo`** — HOC-обёртка, которая пропускает ререндер, если пропсы **поверхностно равны** предыдущим. Разница: HOC — архитектурный приём (обёртка ради логики), `memo` — оптимизация рендера (обёртка ради сравнения пропсов). Современная альтернатива HOC — **хуки** и **композиция через `children`**; вместо `memo` вручную часто применяют **React Compiler**.

## Подробно

### HOC

```jsx
function withAuth(WrappedComponent) {
  return function AuthGuard(props) {
    const user = useUser();
    if (!user) return <Redirect to="/login" />;
    return <WrappedComponent {...props} user={user} />;
  };
}

const ProfilePage = withAuth(Profile);
```

Правила и проблемы:
- **Не мутировать** входной компонент, возвращать новый.
- **`displayName`** для отладки: `AuthGuard.displayName = \`withAuth(${WrappedComponent.displayName ?? WrappedComponent.name})\``.
- **Не объявлять HOC внутри `render`** — каждый рендер новая компонентная функция → размонтирование поддерева.
- Проблема «wrapper hell»: `withAuth(withTheme(withRouter(...)))`.
- Пропсы-«прокидывания» засоряют интерфейс; TypeScript сложнее.
- **Статические свойства** нужно копировать (`hoist-non-react-statics`).

Классика: `connect` из react-redux, `withRouter`, `styled-components`.

### `React.memo`

```jsx
const Row = React.memo(function Row({ item, onSelect }) {
  return <li onClick={() => onSelect(item.id)}>{item.title}</li>;
});

// Можно сравнить вручную
const Row = React.memo(RowInner, (prev, next) => prev.item.id === next.item.id);
```

- **Поверхностное** сравнение (`shallow equal`) по ключам пропсов.
- Не помогает, если пропсы — новые объекты/функции каждый рендер.
- **Не заменяет** мемоизацию: нужны `useMemo`/`useCallback` у родителя.
- В dev-режиме Profiler покажет причину ререндера.

### HOC vs hooks

| | HOC | Хуки |
|---|---|---|
| Переиспользование | оборачивает компонент | функция внутри |
| Структура | дополнительный узел, wrapper hell | плоская |
| Пропсы | инъекция через props | возврат значений |
| Статики/DOM ref | сложно | тривиально |
| Тестируемость | компонент + обёртка | чистый хук |

Сейчас HOC оправдан для «оборачивания поведения, которое должно быть в дереве»: Error Boundary, Suspense-обёртки, `connect`-подобные библиотеки. Логику — в хуки.

### `memo` и примитивы

Если родитель передаёт примитивы (`string`, `number`, `boolean`) — `memo` работает без мемоизации. Если объекты/массивы/функции — нужен `useMemo`/`useCallback` **или** React Compiler. Как вариант — сравнение «по значению»:

```jsx
React.memo(Row, (a, b) => a.id === b.id && a.title === b.title);
```

> [!warning] Частая ошибка
> `memo` без стабильных пропсов: родитель передаёт `style={{...}}` или inline `onClick={() => {}}` — ререндер будет всегда, `memo` бесполезен. На собеседовании это стоит проговорить явно, а не просто «обернуть в memo».

## Связанные вопросы

- [[Почему memo не всегда работает]]
- [[Избежание лишних ререндеров]]
- [[useMemo и useCallback]]

## Ресурсы

- react.dev memo: https://react.dev/reference/react/memo
- react.dev HOC: https://react.dev/reference/react/Component#higher-order-components
