---
type: careers/interview
section: Браузер и сети
topic: Платформа
status: ready
created: 2026-11-16
tags: [interview, browser, web-platform]
---

# Что такое Web API

## Краткий ответ (30 секунд)

**Web API** — набор интерфейсов, которые предоставляет браузер (или другое окружение — Node, Deno) поверх JavaScript. Сам язык (ECMAScript) знает про объекты, массивы, промисы, но не про `window`, `document`, `fetch`, `localStorage`, `setTimeout`, `navigator`. Это «вторая половина» платформы: спецификации WHATWG/W3C/TC39 описывают, что движок должен дать скрипту. Web API — не библиотеки, а встроенные интерфейсы, которые браузер реализует на C++/Rust и оборачивает в JS-объекты.

## Подробно

### Что входит

| Категория | Примеры |
|---|---|
| DOM | `document`, `Element`, `NodeList`, события |
| Сеть | `fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource`, `Beacon` |
| Хранение | `localStorage`, `IndexedDB`, `CacheStorage`, cookies |
| Графика | Canvas 2D, WebGL, WebGPU, SVG |
| Локальная БД и файлы | OPFS, File System Access API, `Blob`, `File` |
| Медиа | `MediaDevices`, `Recorder`, `AudioContext`, WebRTC |
| Параллельность | `Worker`, `SharedArrayBuffer`, `Atomics` |
| Асинхронность | `setTimeout`, `requestAnimationFrame`, `queueMicrotask` |
| Мониторинг | `PerformanceObserver`, `PerformanceTiming` |
| Устройства | `Geolocation`, `Notification`, `Clipboard`, `Vibration` |
| Безопасность | `crypto.subtle`, `CredentialManagement` |

### Как устроено

- **ECMAScript** определяет, как работает код: типы, `this`, замыкания.
- **Web IDL** определяет, как выглядит объект из JS: `window.fetch(input, init)`, метод с параметрами и `Promise`-ом.
- **Event Loop и host-объекты** — спецификация HTML (WHATWG): очереди, таймеры, `rAF`, `postMessage`.
- **Реализация** — Chromium Blink (C++), WebKit, Gecko. Каждый интерфейс мостится в JS: вызов из JS → реализация в движке → результат/событие обратно.

### Важные следствия

1. **Не всё одинаково в разных браузерах.** `requestIdleCallback` отсутствовал в Safari долго, `showSaveFilePicker` — только Chromium. Поэтому проверяют [Baseline](https://web-platform-dx.github.io/web-features-explorer/).
2. **Web API зависят от окружения**: в Node есть `fetch`, `AbortController`, `URL`, `crypto`, но нет `document`/`localStorage`; в worker нет DOM.
3. **Web API часто работает через Promise и события** — это и есть пример «асинхронного программирования под браузер».
4. **Не всё в `window` — стандарт**: глобальные объекты могут быть добавлены расширениями или библиотеками. Для проверки API смотрите `MDN`/Baseline, а не `typeof`.

### Частый вопрос: «чем отличается JavaScript от Web API»

> JS — язык (спецификация ECMAScript). Web API — окружение (браузер), где этот язык исполняется. `Array.prototype.map` — язык, `document.querySelector` — браузер.

### Проверка поддержки

```js
if ("IntersectionObserver" in window) {
  // используем observer
} else {
  // fallback через scroll
}

// или
const hasOpfs = "storage" in navigator && "getDirectory" in navigator.storage;
```

> [!tip] Формулировка
> «Web API — интерфейсы браузера: DOM, fetch, storage, Workers. Они описаны в спецификациях WHATWG/W3C, а не в ECMAScript. Поэтому браузеры реализуют их по-разному и поддерживают не всегда одинаково».

## Связанные вопросы

- [[Service Worker и Web Worker]]
- [[WebSocket и WebRTC]]
- [[Где хранить данные на фронтенде]]

## Ресурсы

- MDN Web APIs: https://developer.mozilla.org/ru/docs/Web/API
- Web platform features explorer: https://web-platform-dx.github.io/web-features-explorer/
