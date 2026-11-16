---
type: careers/interview
section: JavaScript
topic: Браузерные API
status: ready
created: 2026-11-16
tags: [interview, javascript, browser-api, observers]
---

# IntersectionObserver и ResizeObserver

## Краткий ответ (30 секунд)

`IntersectionObserver` асинхронно сообщает, когда элемент **пересекается** с другим элементом (обычно с viewport или скролл-контейнером) — используется для ленивой загрузки картинок, появления элементов при скролле, «бесконечной» подгрузки, отслеживания рекламы. `ResizeObserver` сообщает об **изменении размера** элемента (в том числе при изменении размеров контейнера) — нужен для адаптивных виджетов, текста в ограниченной области, ресайза Canvas. Оба работают асинхронно через микрозадачи/кадр, не вызывают scroll-listener-лагов и не требуют `getBoundingClientRect` в цикле.

## Подробно

### IntersectionObserver

```js
const observer = new IntersectionObserver(
  (entries, obs) => {
    for (const entry of entries) {
      if (entry.isIntersecting) {
        loadImage(entry.target); // ленивая загрузка
        obs.unobserve(entry.target);
      }
    }
  },
  {
    root: null, // viewport
    rootMargin: "200px", // «отступ» для предзагрузки
    threshold: 0.5, // доля видимости
  },
);

document.querySelectorAll("img[data-src]").forEach((img) => observer.observe(img));
```

- **`root`** — контейнер/`null` для viewport.
- **`rootMargin`** — расширение/сужение области срабатывания (например, `"200px"` — начать загрузку раньше).
- **`threshold`** — от 0 до 1 или массив `[0, 0.25, 0.5]`.
- **`entry`** — `{ target, intersectionRatio, isIntersecting, boundingClientRect, intersectionRect }`.
- **`observer.unobserve`** — перестать наблюдать (например, после загрузки).

Типичные кейсы:
- превью в ленте, подгружаются только при прокрутке;
- «появился в кадре» — появление с анимацией;
- прогресс чтения статьи / посты;
- бесконечная подгрузка: observer на «sentinel»-элементе в конце списка.

### Почему не scroll-listener

```js
// ❌ скролл-обработчик руками и getBoundingClientRect в каждом событии
window.addEventListener("scroll", () => {
  const rect = el.getBoundingClientRect();
  if (rect.top < window.innerHeight) load();
});
```

Проблемы: синхронные layout-чтения в частом обработчике, лаги, много кода. `IntersectionObserver` — асинхронный, батчится и не блокирует основной поток.

### ResizeObserver

```js
const resizeObserver = new ResizeObserver((entries) => {
  for (const entry of entries) {
    const { width, height } = entry.contentRect;
    canvas.width = width;
    canvas.height = height;
  }
});

resizeObserver.observe(widthContainer);
```

- Наблюдает **сам элемент**, а не viewport.
- Срабатывает после layout — если в колбэке менять размер элемента, можно получить **«ResizeObserver loop»**; браузер отдаст предупреждение и пропустит кадр.
- `entry.borderBoxSize` / `entry.contentBoxSize` — точные размеры для CSS-логики (с учётом `box-sizing`).
- Для изображений и видео есть `entry.devicePixelContentBoxSize` — точные пиксели устройства.

Типичные кейсы:
- компонент, который меняет layout по ширине контейнера (не по ширине окна);
- Canvas, который должен занимать родителя;
- виртуальные списки (пересчёт высоты).

### Сравнение с другими API

| API | Наблюдает | Может ли сработать без изменений окна |
|---|---|---|
| `ResizeObserver` | размер элемента | да (контент/шрифт изменились) |
| `IntersectionObserver` | видимость | да (скролл, `rootMargin`) |
| `MutationObserver` | DOM-изменения | да |
| `PerformanceObserver` | perf-записи | да |

### Сборка / отложенное

```js
const container = document.querySelector(".items");
const observer = new IntersectionObserver(([entry]) => {
  if (entry.isIntersecting) loadMore();
}, { rootMargin: "300px" });

observer.observe(container); // «sentinel» в конце списка
```

## Связанные вопросы

- [[Event Loop]]
- [[requestAnimationFrame и requestIdleCallback]]
- [[Оптимизация приложения и метрики]]

## Ресурсы

- MDN IntersectionObserver: https://developer.mozilla.org/ru/docs/Web/API/IntersectionObserver
- MDN ResizeObserver: https://developer.mozilla.org/ru/docs/Web/API/ResizeObserver
