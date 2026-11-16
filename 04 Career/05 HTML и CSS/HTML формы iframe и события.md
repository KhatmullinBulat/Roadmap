---
type: careers/interview
section: HTML и CSS
topic: HTML
status: ready
created: 2026-11-16
tags: [interview, html, dom, events]
---

# HTML: формы, iframe, хранение, события

## Краткий ответ (30 секунд)

Это набор «мелких» вопросов HTML, которые часто задают вперемешку: **формы** (`FormData`, валидация, `name`), **`iframe`** (sandbox, общение через `postMessage`, CORS/CLS), **`data-*`** (метаданные в DOM), **скрытые состояния** (`hidden`, `display:none`, `aria-hidden`), **drag-and-drop**, **streaming/`<video>`**, **`template`/`slot`**. Знать надо уверенно: `name` и `FormData`, `sandbox` и `allow`, безопасность `postMessage`, и разницу `hidden`/`display:none`/`aria-hidden`.

## Подробно

### Формы

```html
<form id="f">
  <input name="email" type="email" required />
  <input name="tags" value="a" />   <!-- без name не попадёт в FormData -->
  <button type="submit">Отправить</button>
</form>
<script>
  const fd = new FormData(f);
  fd.get("email");
</script>
```

- **`name`** — ключ в `FormData` (подробнее — [[Атрибут name у input]]).
- Нативная валидация: `required`, `type`, `pattern`, `min/max`, `setCustomValidity`; `form.checkValidity()`, `form.reportValidity()`; CSS `:invalid`, `:user-invalid` (лучше — не краснить пустые поля до ввода).
- `type="submit"` по умолчанию у `<button>` внутри формы.
- `enctype="multipart/form-data"` для файлов; `accept` — фильтр выбора.
- `formData(form, submitter)` — включает кнопку, инициировавшую отправку.

### iframe

```html
<iframe src="https://player.example.com"
        sandbox="allow-scripts allow-same-origin"
        allow="autoplay; fullscreen"
        loading="lazy"
        title="Видео"></iframe>
```

- **`sandbox`** — ограничивает; без `allow-scripts` JS не выполнится; `allow-scripts` + `allow-same-origin` вместе ослабляют песочницу (если src same-origin, фрейм получит доступ к родителю).
- **`allow`** — permissions policy (камера, микрофон, fullscreen, autoplay, геолокация).
- `loading="lazy"` — отложить загрузку (но осторожно с видимыми iframe).
- Не забывать `title` — доступность.
- **X-Frame-Options/`frame-ancestors`** — кто может встраивать.
- CLS: фиксируйте размеры/`aspect-ratio`.

### `postMessage` — общение между окнами

```js
// родитель
iframe.contentWindow.postMessage({ type: "ping" }, "https://player.example.com");

window.addEventListener("message", (e) => {
  if (e.origin !== "https://player.example.com") return; // обязательная проверка
  console.log(e.data);
});
```

- **Всегда проверять `event.origin`** и `event.source`.
- `targetOrigin` — не `*`, если можно указать конкретный.
- Данные — structured clone (проверять), это не «сериализация в безопасный формат».

### `data-*`

```html
<div data-id="42" data-state="open"></div>
<script>
  el.dataset.id;    // "42"
  el.dataset.state; // "open"
</script>
```

- Для JS-разметки, не для стилей (хотя `[data-state="open"]` — нормально).
- `dataset` — только camelCase-ключи, значения строки.

### Скрытие: три разных вещи

| Способ | Видно | В дереве доступности | Можно фокусировать |
|---|---|---|---|
| `hidden`/`display:none` | нет | нет | нет |
| `visibility:hidden` | нет | нет | нет |
| `opacity:0` | нет | **да** | **да** |
| `aria-hidden="true"` | да | нет | **да** (ловушка) |
| `clip: rect(0 0 0 0)` (sr-only) | визуально нет | да | да (для скринридера) |

### `<template>`, `<slot>`, `<dialog>`

- `<template>` — инертная разметка; клонируется в JS.
- `<slot>` — для Web Components (см. [[Что такое Shadow DOM]]).
- `<dialog>` — нативная модалка: `showModal()`, `::backdrop`, `Esc` закрывает, ловушка фокуса. **Это правильный ответ на «чем делать модалку»**.

### Частые подвохи

- `<img alt>` без значения — декоративная картинка (не забыть `alt=""`).
- `autocomplete="off"` на пароле — браузеры могут игнорировать.
- `<table>` для вёрстки — антипаттерн; для данных — с `<caption>`, `<th scope>`.
- `htmlFor`/`className` — только в JSX, в HTML — `for`/`class`.

> [!tip] Как отвечать
> «`name` — ключ FormData. iframe — sandbox и проверка origin в `message`. Скрытие: `display:none` (нет в a11y-дереве), `aria-hidden` (видно, но не озвучивается), sr-only — видно скринридеру. Модалка — нативный `<dialog>`, а не div».

## Связанные вопросы

- [[Атрибут name у input]]
- [[Семантическая верстка]]
- [[Доступность]]

## Ресурсы

- MDN FormData: https://developer.mozilla.org/ru/docs/Web/API/FormData
- MDN `postMessage`: https://developer.mozilla.org/ru/docs/Web/API/Window/postMessage
