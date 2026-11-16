---
type: careers/interview
section: JavaScript
topic: Асинхронность
status: ready
created: 2026-11-16
tags: [interview, javascript, event-loop, concurrency]
---

# Event Loop

## Краткий ответ (30 секунд)

JavaScript — **однопоточный**, но асинхронный. **Event Loop** — механизм, который после опустошения стека вызовов забирает готовую работу из очередей и выполняет её. Порядок: сначала **микрозадачи** (промисы, `queueMicrotask`, `MutationObserver`), затем **одна макрозадача** (`setTimeout`, `setInterval`, `MessageChannel`, сетевые/`UI`-события), после чего снова микрозадачи — и так по кругу. **`requestAnimationFrame`** — отдельные колбэки отрисовки, **`requestIdleCallback`** — выполнение в простое. Микрозадачи, которые создают новые микрозадачи, могут «заморить» отрисовку и таймеры.

## Подробно

### Разные слушатели

В **браузере** очередь задач — это список макрозадач + очередь микрозадач. Они живут в главном потоке страницы (вкладке), а не в процессе браузера. **Node.js** использует `libuv` с фазами (`timers`, `poll`, `check`, `close callbacks`), и `process.nextTick` там — свой механизм, который идёт **раньше** `Promise`-микрозадач.

### Порядок в браузере

```js
console.log("1");

setTimeout(() => console.log("2"), 0);
queueMicrotask(() => console.log("3"));
Promise.resolve().then(() => console.log("4"));
queueMicrotask(() => console.log("5"));

console.log("6");

// 1, 6, 3, 4, 5, 2
```

- Все микрозадачи выполняются **полностью** перед следующим шагом макрозадач.
- Каждый `then` — это отдельная микрозадача.
- `await` — тоже микрозадача: продолжение после `await` ставится в очередь микрозадач.

```js
async function f() {
  console.log("start");
  await null;        // → микрозадача
  console.log("after"); // выполнится после синхронного кода
}

console.log("before");
f();
console.log("sync");

// before, start, sync, after
```

### Способы создать макрозадачу

| Способ | Особенности |
|---|---|
| `setTimeout(fn, 0)` | в одном цикле; таймер минимум ~4 мс после 5 вложенных |
| `setInterval` | повторяющаяся |
| `MessageChannel().port.postMessage` | макрозадача без таймера, часто быстрее `setTimeout` |
| `setImmediate` | Node.js; `check`-фаза |
| события UI | клики, `scroll`, `network` |
| `window.postMessage` | межоконное сообщение |

### Способы создать микрозадачу

| Способ | Особенности |
|---|---|
| `Promise.resolve().then(fn)` | базовый, самый частый |
| `queueMicrotask(fn)` | явный API, без промиса |
| `await` | продолжение после `await` |
| `MutationObserver` | **batch**: колбэк после всех синхронных мутаций |
| `process.nextTick` | Node.js, идёт до промисов |

```js
queueMicrotask(() => console.log("microtask"));
```

### `requestAnimationFrame`

`rAF` — отдельная очередь «перед следующей отрисовкой». Её цель — синхронизировать анимации с частотой экрана (обычно 60 Гц → ~16.7 мс). Не микрозадача и не «обычная» макрозадача: браузер сам решает, когда кадру нужно обновляться. `rAF` вызывается перед отрисовкой, пока вкладка активна (в фоновой может не вызываться). См. [[requestAnimationFrame и requestIdleCallback]].

### `requestIdleCallback`

`rIC` — выполнение в свободное время между кадрами. Если задача не уложилась в период, `deadline.timeRemaining()` подскажет, что осталось; короткие задачи можно разбить и продолжить. Приоритет ниже остальных, и **не обязательно вызовется**; поддержка в браузерах неравномерна.

### «Сначала микрозадачи» — практический вывод

```js
// Плохо: рендер «зависнет», потому что микрозадачи бесконечны
Promise.resolve().then(function loop() {
  Promise.resolve().then(loop); // бесконечная цепочка
});

// Хорошо: периодическая работа через макрозадачу
function poll() {
  doChunk();
  setTimeout(poll, 0); // даёт браузеру отрисовать кадр
}
```

### Как это видно в DevTools

Performance → Main thread: микрозадачи (Promise/queueMicrotask) — `Run Microtasks`, `Timer fired`, `Function call`. `Render`/`Commit` — кадр. Паттерн «долгая Майн-задача + длинный Run Microtasks» — признак, что решение синхронное.

> [!warning] Важное уточнение
> Цикл без выхода **не** отдаёт кадр — ни `rAF`, ни `setTimeout` не выполнятся, пока стек не опустеет. `while (true) {}` не даёт рендеру шанса.

## Таблица приоритетов (упрощённо)

1. Синхронный код (call stack)
2. Микрозадачи (`Promise`, `queueMicrotask`)
3. `requestAnimationFrame` / `style`-пересчёт / отрисовка
4. Макрозадачи (`setTimeout`, событие)
5. `requestIdleCallback`
6. Worker — другой поток, там свой цикл

## Связанные вопросы

- [[Что такое Promise]]
- [[requestAnimationFrame и requestIdleCallback]]
- [[Как работает Garbage Collection в браузере]]
- [[Почему memo не всегда работает]] — батчинг и приоритеты

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/JavaScript/Event_loop
- HTML spec: https://html.spec.whatwg.org/multipage/webappapis.html#event-loops
