---
type: careers/interview
section: JavaScript
topic: Асинхронность
status: ready
created: 2026-11-16
tags: [interview, javascript, fetch, abort]
---

# AbortController

## Краткий ответ (30 секунд)

`AbortController` — стандартный способ **отменить** асинхронную операцию, которая поддерживает `signal`. `controller.signal` — это `AbortSignal`; когда вызывают `controller.abort()`, сигнал переходит в состояние `aborted`, и всё, что на него подписано, получает событие `abort` и прерывается. Используется в `fetch` (сетевой запрос), в `addEventListener(..., { signal })` (снятие обработчика), в таймерах через API-обёртки, в `ReadableStream`, WebRTC. Это не «отмена промиса» в языке — это **протокол кооперативной отмены**.

## Подробно

### Отмена fetch

```js
const controller = new AbortController();

const promise = fetch("/api/items", { signal: controller.signal });

// через 300 мс
setTimeout(() => controller.abort(), 300);

try {
  const res = await promise;
} catch (err) {
  if (err.name === "AbortError") {
    console.log("запрос отменён");
  } else {
    throw err;
  }
}
```

`fetch` не «убивает» промис — он **reject'ит** его с `DOMException` с `name === "AbortError"`.

### Таймаут

```js
const res = await fetch(url, {
  signal: AbortSignal.timeout(5000),
});
```

`AbortSignal.timeout(ms)` — стандартный таймаут-сигнал. `AbortSignal.any([signalA, signalB])` — объединить сигналы (отмена при любом из них).

### Снятие обработчиков

```js
const controller = new AbortController();

const onScroll = () => {};
window.addEventListener("scroll", onScroll, { signal: controller.signal });
window.addEventListener("resize", onResize, { signal: controller.signal });

// Один вызов — снимает все обработчики, подписанные с этим сигналом
controller.abort();
```

Это современная замена «складывать все removeEventListener вручную». В React часто комбинируют с `useEffect`:

```js
useEffect(() => {
  const controller = new AbortController();
  fetch("/api/data", { signal: controller.signal })
    .then((r) => r.json())
    .then(setData)
    .catch((e) => { if (e.name !== "AbortError") setError(e); });

  return () => controller.abort(); // при unmount / смене зависимостей
}, [id]);
```

### Свои API

```js
function withAbort(signal, task) {
  return new Promise((resolve, reject) => {
    const onAbort = () => reject(new DOMException("Aborted", "AbortError"));
    if (signal.aborted) return onAbort();

    signal.addEventListener("abort", onAbort, { once: true });
    task().then(resolve, reject).finally(() => {
      signal.removeEventListener("abort", onAbort);
    });
  });
}
```

### `abort(reason)`

```js
controller.abort(new Error("пользователь ушёл"));
```

Современный `abort(reason)` позволяет передать причину; она попадёт в `signal.reason`.

### Чего он не делает

- Не отменяет **уже завершённый** запрос.
- Не прерывает вычисление в основном потоке — для тяжёлой математики нужен Worker.
- Отмена `fetch` — это не «отмена на сервере»: запрос часто уже дошёл, просто клиент не ждёт ответ.
- Не автоматически снимает подписки, созданные без `{ signal }`.

> [!tip] Как отвечать
> «`AbortController` — кооперативная отмена: даём `signal`, при `abort()` всё, что подписалось, получает `abort`. Удобен в `fetch` и `addEventListener`, а также в `useEffect` cleanup. Отмена `fetch` приходит как `AbortError` в `catch`».

## Связанные вопросы

- [[Что такое Promise]]
- [[Методы Promise]]
- [[requestAnimationFrame и requestIdleCallback]]

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/API/AbortController
