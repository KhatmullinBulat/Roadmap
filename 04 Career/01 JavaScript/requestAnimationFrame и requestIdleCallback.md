---
type: careers/interview
section: JavaScript
topic: Браузерные API
status: ready
created: 2026-11-16
tags: [interview, javascript, browser-api, performance]
---

# requestAnimationFrame и requestIdleCallback

## Краткий ответ (30 секунд)

`requestAnimationFrame(cb)` планирует колбэк на **следующий кадр**, перед отрисовкой браузера (обычно раз в 16.7 мс при 60 Гц); колбэк получает высокоточный `timestamp`. Используется для анимаций, скролла, визуальных измерений — всё, что должно успеть *до* paint. `requestIdleCallback(cb)` планирует работу в **свободное время** между кадрами, с `deadline.timeRemaining()`; может не выполниться вовсе, приоритет ниже всего. Используется для некритичной фоновой работы: аналитика, предзагрузка, разбор данных.

## Подробно

### requestAnimationFrame

```js
function animate(time) {
  const x = Math.min((time / 10) % 400, 400);
  box.style.transform = `translateX(${x}px)`;
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

- Вызывается **перед paint**, синхронно с текущим `style`/layout браузера.
- Если вкладка скрыта, колбэк **не вызывается** (экономия энергии), поэтому `setInterval` для анимаций — антипаттерн.
- Возвращает `id`, отменяется `cancelAnimationFrame(id)`.
- `timestamp` удобно использовать для расчёта прогресса вместо `Date.now()` — плавнее и честнее.

### Что происходит в кадре

```
JS → style → layout → paint → composite
     ↑
  ваши rAF-колбэки выполняются здесь
```

Если вы меняете стили в `rAF`, браузер успевает пересчитать layout до того же кадра. Если делаете это в `setTimeout`, кадр может быть пропущен.

### requestIdleCallback

```js
function work(deadline) {
  while (deadline.timeRemaining() > 2) {
    processNextItem();
  }
  if (hasMore) requestIdleCallback(work);
}

requestIdleCallback(work, { timeout: 1000 }); // timeout — гарантировать вызов
```

- `deadline.timeRemaining()` — сколько мс осталось до конца «окна бездействия» (обычно стоит 50 мс).
- `{ timeout }` — заставит вызвать, даже если браузер не найдёт свободного времени.
- Может не выполниться никогда (например, при непрерывной анимации) — **не годится для критичных операций**.

### Сравнение

| | `requestAnimationFrame` | `requestIdleCallback` |
|---|---|---|
| Когда | перед кадром | пока браузер ждёт |
| Гарантия вызова | да, пока вкладка активна | нет (без `timeout`) |
| Частота | кадр | по усмотрению браузера |
| Применение | анимации, скролл, измерения | аналитика, предзагрузка, тяжёлый парсинг |
| `deadline` | нет, только `timestamp` | да |

### Общая ошибка

```js
// ❌ Бесконечная анимация через setTimeout — пропуски кадров и дёрганье
setInterval(() => move(), 16);
```

Даже `requestAnimationFrame` стоит использовать аккуратно: колбэк должен быть коротким, иначе соберётся длинная задача.

### Отмена

```js
const id = requestAnimationFrame(animate);
cancelAnimationFrame(id);

const idleId = requestIdleCallback(work);
cancelIdleCallback(idleId);
```

> [!tip] Как отвечать
> «`rAF` — синхронизация с кадром: анимации, `scroll`, чтение `getBoundingClientRect` перед стилями. `rIC` — в свободное время, с `timeRemaining()`; если нужна гарантия — `timeout`. Для пользовательской работы всё равно важнее `rAF`, потому что `rIC` может не выполниться».

## Связанные вопросы

- [[Event Loop]]
- [[IntersectionObserver и ResizeObserver]]

## Ресурсы

- MDN `requestAnimationFrame`: https://developer.mozilla.org/ru/docs/Web/API/Window/requestAnimationFrame
- web.dev: https://web.dev/articles/optimize-javascript-execution
