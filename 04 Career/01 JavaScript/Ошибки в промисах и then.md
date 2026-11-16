---
type: careers/interview
section: JavaScript
topic: Асинхронность
status: ready
created: 2026-11-16
tags: [interview, javascript, promises, errors]
---

# Обработка ошибок через then

## Краткий ответ (30 секунд)

**Да, можно.** `then(onFulfilled, onRejected)` — второй аргумент это обработчик ошибки, и он ловит rejection **того** промиса, на котором вызван. Но `.catch()` читается лучше и ловит ошибки **всей цепочки до него**, а второй аргумент `then` — только ошибку предыдущего звена: ошибка, брошенная в `onFulfilled`, через второй аргумент этого же `then` **не** ловится.

## Подробно

### Варианты

```js
const p = Promise.reject(new Error("boom"));

// 1) then с двумя аргументами
p.then(
  (v) => handleValue(v),
  (err) => handleError(err), // ловит rejection p
);

// 2) catch
p.catch((err) => handleError(err)); // то же самое

// 3) then(...).catch(...)
p.then((v) => riskyWork(v))
 .catch((err) => handleError(err)); // ловит и rejection p, и throw в riskyWork
```

### Главное различие

```js
// A: второй аргумент then НЕ ловит ошибку из onFulfilled
p.then(
  (v) => { throw new Error("в обработчике"); },
  (err) => console.log("не поймает это"), // ❌
);

// B: catch ловит
p.then((v) => {
  throw new Error("в обработчике");
}).catch((err) => console.log("поймает это")); // ✅
```

Это самый частый вопрос к выражению `then(onFulfilled, onRejected)`.

### Забытый return — другая классика

```js
// ❌ ошибка теряется: catch повешен на внешний промис, а не на внутренний
fetch("/api").then(async (res) => {
  await handle(res).catch(() => {}); // внутренний catch только тут
});
```

```js
// ✅ цепочка сохраняется, если вернуть промис
fetch("/api")
  .then((res) => handle(res))
  .catch(onError);
```

Если верны оба — `try/catch` внутри `async`-функции и `.catch` на её вызове — можно получить двойную обработку.

### Что `catch` НЕ ловит

- Ошибки в колбэках **вне** промисов (`setTimeout`, `addEventListener`).
- Синхронные ошибки до создания промиса.
- Ошибки, брошенные внутри `onRejected`, если после нет ещё одного `catch`.

```js
Promise.resolve()
  .then(() => { throw new Error("1"); })
  .catch((e) => { console.log(e.message); throw new Error("2"); })
  .catch((e) => console.log(e.message)); // "2"
```

### finally и ошибки

```js
p.finally(() => hideSpinner());
```

`finally` не заменяет ошибку, если только сам не бросит. Ошибка из `finally` перекрывает исходную.

### Показать ошибку вне промиса

```js
window.addEventListener("unhandledrejection", (e) => {
  console.error("Необработанный rejection", e.reason);
});
```

## Связанные вопросы

- [[Методы Promise]]
- [[Promise и async await]]
- [[Что такое Promise]]

## Ресурсы

- MDN `then`: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Promise/then
