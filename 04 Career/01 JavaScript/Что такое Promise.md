---
type: careers/interview
section: JavaScript
topic: Асинхронность
status: ready
created: 2026-11-16
tags: [interview, javascript, promises]
---

# Promise

## Краткий ответ (30 секунд)

`Promise` — объект-обёртка над результатом асинхронной операции с тремя состояниями: **pending → fulfilled** или **pending → rejected**. Переход однократный и необратимый. Результат и состояние — внутренние слоты `[[PromiseState]]` и `[[PromiseResult]]`; снаружи доступны только `then`/`catch`/`finally` и статические методы. `Promise` — это **не вычисление, а событие**: работа запускается сразу в executor при создании, а не при `then`.

## Подробно

### Создание

```js
const promise = new Promise((resolve, reject) => {
  // executor вызывается СИНХРОННО
  doAsync((err, data) => (err ? reject(err) : resolve(data)));
});
```

- **Executor** вызывается синхронно в конструкторе.
- `resolve(value)` выполняет; если `value` — тоже промис, промис «подписывается» на него (assimilation).
- `reject(reason)` отклоняет.
- `throw` внутри executor равносилен `reject`.
- Повторные `resolve`/`reject` игнорируются.

### Порядок выполнения

```js
const p = new Promise((res) => {
  console.log("executor"); // 1
  res(42);
  res(43); // игнорируется
});

p.then((v) => console.log("then", v)); // 3
console.log("synchronous");            // 2
```

Колбэки `then` **всегда** асинхронны — они попадают в очередь микрозадач, даже если промис уже исполнен.

### Тогда и цепочки

```js
fetch("/user")
  .then((res) => res.json())
  .then((user) => fetch(`/orders/${user.id}`))
  .then((res) => res.json())
  .catch((err) => reportError(err))
  .finally(() => hideSpinner());
```

- `then` возвращает **новый** промис — поэтому возможна цепочка.
- Возврат значения из `then` → fulfilled этим значением.
- `throw` в `then` → rejected промис.
- Если `then` возвращает промис, он «сплющивается»: `Promise<Promise<T>>` не появляется.

### Внутренние слоты

| Слот | Что хранит |
|---|---|
| `[[PromiseState]]` | pending / fulfilled / rejected |
| `[[PromiseResult]]` | значение или причина |
| `[[PromiseFulfillReactions]]` | список `then`-реакций |
| `[[PromiseRejectReactions]]` | список обработчиков ошибок |
| `[[PromiseIsHandled]]` | была ли ошибка обработана (иначе `unhandledrejection`) |

Подробнее про нотацию — [[Внутренние слоты]].

### Свойства, которые стоит назвать

- **Не отменяемый**: `fetch` с `AbortController` — это внешняя отмена, а не отмена промиса.
- **Eager**: асинхронная операция начинается сразу в executor, а не в `then`.
- **Immutable outcome**: нельзя «перерешать» промис.
- **Microtask**: реакции всегда выполняются в очереди микрозадач.

### Необработанные ошибки

```js
const p = Promise.reject(new Error("boom"));
// ошибочный промис без обработчика:

setTimeout(() => console.log("x"), 0);
// → unhandledrejection в браузере
```

`window.addEventListener("unhandledrejection", ...)` и `process.on("unhandledRejection", ...)` в Node. Если обработчик добавится позже (в следующем тике), может сработать `unhandledrejection` — это частый источник мигающих ошибок.

> [!tip] Как отвечать
> «Promise — объект статуса асинхронной операции: pending/fulfilled/rejected и результат. Он не отменяемый и не изменяемый после исполнения. Реакции `then` идут в микрозадачи, ошибки без `catch` всплывают как `unhandledrejection`».

## Связанные вопросы

- [[Методы Promise]]
- [[Promise и async await]] — синтаксический сахар над промисами
- [[Event Loop]]
- [[AbortController]]

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Promise
- Спецификация: https://tc39.es/ecma262/#sec-promise-objects
