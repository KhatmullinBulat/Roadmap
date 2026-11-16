---
type: careers/interview
section: JavaScript
topic: Асинхронность
status: ready
created: 2026-11-16
tags: [interview, javascript, promises, async-await]
---

# Promise и async/await

## Краткий ответ (30 секунд)

`async/await` — **синтаксический сахар** над промисами: `await` приостанавливает `async`-функцию, пока промис не исполнится, и «распаковывает» его результат. Обычный `then` — это явная работа с состоянием и цепочками; `await` даёт **линейный, синхронный по виду код** и обычный `try/catch` для ошибок. Разница на практике: в `await` легко случайно **сериализовать** независимые запросы (последовательные вместо параллельных), а в `then` легко забыть `return`/обработку ошибок. Оба варианта используют один и тот же механизм: колбэки промиса, попадающие в микрозадачи.

## Подробно

### Один и тот же код в двух стилях

```js
// then
function loadUser(id) {
  return fetch(`/api/users/${id}`)
    .then((res) => {
      if (!res.ok) throw new Error(res.statusText);
      return res.json();
    })
    .then((user) => user.name)
    .catch((err) => {
      logError(err);
      throw err;
    });
}
```

```js
// async/await
async function loadUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(res.statusText);
    const user = await res.json();
    return user.name;
  } catch (err) {
    logError(err);
    throw err;
  }
}
```

### Что на самом деле делает async

```js
async function f() {
  return 1;
}
console.log(f()); // Promise {<fulfilled>: 1}
```

`async`-функция **всегда** возвращает промис. Возвращённое значение попадает в `resolve`, `throw` — в `reject`. `await` перед непромисом тоже работает: он оборачивает значение в `Promise.resolve`.

### Parallelism — самый частый вопрос

```js
// Последовательно: 2 × время запроса
async function sequential() {
  const a = await fetchA(); // ждём
  const b = await fetchB(); // потом ждём
  return [a, b];
}

// Параллельно: одно время
async function parallel() {
  const [a, b] = await Promise.all([fetchA(), fetchB()]);
  return [a, b];
}
```

Даже в параллельном варианте `Promise.all` **стартует оба** промиса сразу, а `await` ждёт оба. Это главное отличие от `then`-цепочки:

```js
Promise.all([fetchA(), fetchB()]).then(([a, b]) => {});
```

### try/catch и ошибки

```js
try {
  const data = await fetchData();
} catch (err) {
  // ловит и sync-исключения, и rejection
}

// Без try/catch:
async function f() {
  const data = await fetchData(); // если fetchData rejected — функция станет rejected
}
```

`try/catch` ловит и синхронные исключения, и rejection'ы `await`. Он **не** ловит ошибки в промисах, которые вы не ожидали: `fetchData();` без `await` — rejection уйдёт в `unhandledrejection`.

### Возврат и finally

- `async function` → `Promise<T>`.
- `return await` vs `return`: `return promise` тоже дожидается, но `finally` в `async` без `await` может выполниться раньше — в `try { return p } finally {}` лучше `return await p`, чтобы `finally` дождался результата.
- `catch` внутри `async` требует `await` для промиса; `catch` в `then`-цепочке — это метод.

### Сравнение

| Критерий | `then` | `async/await` |
|---|---|---|
| Читаемость | явные цепочки | линейный код |
| Параллельность | `Promise.all` вручную | `Promise.all` + `await` |
| Ошибки | `.catch()` | `try/catch` |
| Легко нарваться на | забытый `return` | последовательное `await` |
| Используется в | библиотеках, старом коде | прикладном коде |

### Когда `then` уместнее

- Короткая цепочка из одного шага.
- Функциональный стиль, когда результат идёт в `Promise.all`.
- Использование промиса без его ожидания (fire-and-forget).
- API, где промис создаётся фабрикой (библиотеки).

> [!tip] Главная формулировка
> «`async/await` — это синтаксис над промисами: `async` возвращает промис, `await` приостанавливает функцию до исполнения. На практике разница в том, как легко ошибиться: потерять `return` в `then` или сериализовать независимые `await`».

## Связанные вопросы

- [[Что такое Promise]]
- [[Методы Promise]]
- [[Event Loop]]
- [[Ошибки в промисах и then]]

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Statements/async_function
