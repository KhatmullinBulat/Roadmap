---
type: careers/interview
section: JavaScript
topic: Асинхронность
status: ready
created: 2026-11-16
tags: [interview, javascript, promises]
---

# Методы Promise

## Краткий ответ (30 секунд)

`then` — обработка успеха/провала и продолжение цепочки; `catch` — то же для ошибки; `finally` — побочный эффект в обоих исходах без изменения результата. Статические: `all` — ждёт **все**, падает по первому reject; `race` — первый **завершившийся** (успех или ошибка); `any` — первый **успешный**, падает `AggregateError`, если все провалились; `allSettled` — ждёт всех и возвращает статус каждого; `resolve`/`reject` — создание готового промиса; `withResolvers` — отложенный resolve/reject.

## Подробно

### Инстансные методы

```js
p.then(onFulfilled, onRejected);
p.catch(onRejected);
p.finally(onSettled);
```

- **`then`** возвращает новый промис. Можно передать второй аргумент (`onRejected`) — это то же, что `then(undefined, onRejected)`.
- **`catch`** — это `then(undefined, onRejected)`.
- **`finally`** не получает результат и не меняет его — удобно для `hideSpinner()`, закрытия соединения. Если `finally` бросит — ошибка заменит исходную.

### Статические методы

#### `Promise.all(iterable)`

Ждёт все. Результат — массив значений **в исходном порядке**. Отклоняется по первому reject (остальные промисы продолжат работать, их результаты будут проигнорированы).

```js
await Promise.all([fetchA(), fetchB()]); // [a, b] или бросает
```

#### `Promise.allSettled(iterable)`

Ждёт все, никогда не отклоняется. Даёт `[{status: "fulfilled", value}, {status: "rejected", reason}]`.

```js
const results = await Promise.allSettled([fetchA(), fetchB()]);
const ok = results.filter((r) => r.status === "fulfilled");
```

#### `Promise.race(iterable)`

Первый завершившийся: победа или провал — что раньше. Классический таймаут:

```js
const withTimeout = (promise, ms) =>
  Promise.race([
    promise,
    new Promise((_, reject) => setTimeout(() => reject(new Error("timeout")), ms)),
  ]);
```

#### `Promise.any(iterable)`

Первый **успешный**. Если все провалились — `AggregateError` с `.errors`. Идеален для «первый доступный CDN/зеркало».

```js
const fromCDN = await Promise.any([
  fetch("https://cdn1.example/file"),
  fetch("https://cdn2.example/file"),
]);
```

#### `Promise.resolve(v)` / `Promise.reject(e)`

Создают уже исполненный/отклонённый промис. `Promise.resolve(thenable)` ассимилирует thenable.

#### `Promise.withResolvers()` (ES2024)

Возвращает `{ promise, resolve, reject }` — удобно, когда разрешение приходит из внешнего колбэка:

```js
const { promise, resolve } = Promise.withResolvers();
button.onclick = () => resolve("clicked");
const result = await promise;
```

### Сводная таблица

| Метод | Ждёт | Первый reject | Первый успех | Результат |
|---|---|---|---|---|
| `all` | все | бросает сразу | — | `T[]` |
| `allSettled` | все | никогда | — | `SettledResult[]` |
| `race` | первый любой | может | может | значение/ошибка первого |
| `any` | — | игнорирует | побеждает | значение; `AggregateError` если все упали |

### Частые подвохи

```js
// 1. all с пустым массивом — сразу fulfilled с []
await Promise.all([]); // []

// 2. race с пустым массивом — вечный pending
await Promise.race([]); // никогда не завершится

// 3. all не отменяет остальные
Promise.all([slow(), fastFail()]); // slow() продолжит работать
```

### `then` умеет ловить ошибки

```js
p.then(
  (v) => doWork(v),
  (err) => handle(err), // ловит ошибку самого p
);
```

> [!tip] Формулировка
> «`all` — все или первый провал. `allSettled` — все, никогда не падает. `race` — кто первый, успех или провал. `any` — первый успех, иначе `AggregateError`. `finally` не меняет результат».

## Связанные вопросы

- [[Что такое Promise]]
- [[Promise и async await]]
- [[Ошибки в промисах и then]]
- [[Event Loop]]

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Promise
