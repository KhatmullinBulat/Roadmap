---
type: careers/interview
section: Браузер и сети
topic: Параллельность
status: ready
created: 2026-11-16
tags: [interview, browser, workers, pwa]
---

# Web Worker и Service Worker

## Краткий ответ (30 секунд)

**Web Worker** — отдельный поток для вычислений. У него нет доступа к DOM, но есть `postMessage` и `onmessage` для обмена данными (структурное копирование), `importScripts`/`import` модулей, `fetch`, `IndexedDB`. Родственники: **SharedWorker** (общий для нескольких вкладок) и **Worklet** (аудио/анимации/парсинг). **Service Worker** — *не* заменяет Web Worker: это фоновый «прокси» для сети и кэша, который живёт между навигациями и позволяет офлайн-режим, push-уведомления и фоновую синхронизацию. Он не имеет доступа к DOM, ограничен HTTPS и требует регистрации с активным контролем страницы.

## Подробно

### Web Worker

```js
// main.js
const worker = new Worker(new URL("./worker.js", import.meta.url), { type: "module" });
worker.postMessage({ cmd: "process", items: bigArray });
worker.onmessage = (e) => render(e.data);
```

```js
// worker.js
self.onmessage = ({ data }) => {
  const result = heavyCompute(data.items);
  self.postMessage(result); // копирование по structured clone
};
```

- **Обмен**: структурное копирование (structured clone) — объекты копируются. `Transferable` (ArrayBuffer, OffscreenCanvas, MessagePort) — передаёт владение без копии:

```js
worker.postMessage(buffer, [buffer]); // buffer стал недоступен в main
```

- **`SharedArrayBuffer`** — общая память, нужен `Atomics` и COOP/COEP-заголовки (из-за Spectre).
- **Нельзя:** DOM, `window`, `alert`, `localStorage` (в worker есть `indexedDB`, но не `localStorage`).
- **Зачем:** тяжёлые вычисления (парсинг CSV, шифрование, сортировка), image processing (`OffscreenCanvas`), анимации без main thread, сжатие.
- **Минус:** сообщения копируются, обмен кодом/типами, отладка сложнее.

### Виды worker

| Worker | Особенности |
|---|---|
| Dedicated Worker | один экземпляр на вкладку |
| SharedWorker | один на origin для нескольких вкладок (поддержка в браузерах неровная) |
| Service Worker | прокси/фон между вкладками, события |
| Worklet | аудио (AudioWorklet), paint (CSS Painting), animations |
| Web Worker в Node (`worker_threads`) | серверная версия |

### Service Worker

```js
// страница
navigator.serviceWorker.register("/sw.js");
```

```js
// sw.js
const CACHE = "v1";
self.addEventListener("install", (e) => {
  e.waitUntil(caches.open(CACHE).then((c) => c.addAll(["/", "/index.html", "/app.css"])));
  self.skipWaiting();
});

self.addEventListener("activate", (e) => {
  e.waitUntil(self.clients.claim());
});

self.addEventListener("fetch", (e) => {
  e.respondWith(
    caches.match(e.request).then((hit) => hit || fetch(e.request)),
  );
});
```

- **Жизненный цикл**: `install` → `waiting` → `activate` → `fetch`/`push`/`sync`/`message`.
- **Стратегии кэширования**: cache-first (статика), network-first (динамика), stale-while-revalidate (баланс).
- **Только HTTPS** (localhost — исключение).
- **install** должен уложиться в разумное время: долгие задачи в `waitUntil` могут привести к таймауту и неактивации.
- **Push** — `pushManager.subscribe` + VAPID-ключи на сервере; требуют разрешения и HTTPS.
- **Background Sync** — отложенная отправка (некоторые браузеры).
- **Регистры**: `navigator.serviceWorker.getRegistration()`, проверка обновлений, `update()` вручную.

### Отличия Web Worker vs Service Worker

| | Web Worker | Service Worker |
|---|---|---|
| Время жизни | пока есть ссылка | событиями управляется браузер, может «спать» |
| DOM | нет | нет |
| Назначение | вычисления | перехват сети, офлайн, push |
| Состояние | своё | должно быть в Cache/IndexedDB (может быть убит) |
| Область | страница | origin, все вкладки |
| Регистрация | `new Worker()` | `navigator.serviceWorker.register()` |
| HTTPS | не обязательно | обязательно |

> [!tip] Как отвечать
> «Web Worker — поток для тяжёлого кода, обмен через `postMessage` и structured clone, DOM нет. Service Worker — сетевой прокси с жизненным циклом, кэширует запросы, включает офлайн и push. Первый — для CPU, второй — для сети».

## Связанные вопросы

- [[SPA и PWA]]
- [[Где хранить данные на фронтенде]]
- [[Event Loop]]

## Ресурсы

- MDN Web Workers: https://developer.mozilla.org/ru/docs/Web/API/Web_Workers_API
- MDN Service Worker: https://developer.mozilla.org/ru/docs/Web/API/Service_Worker_API
