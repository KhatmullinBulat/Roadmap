---
type: careers/interview
section: Браузер и сети
topic: Рендеринг
status: ready
created: 2026-11-16
tags: [interview, browser, rendering, performance]
---

# Стадии рендера: layout, paint, compositing

## Краткий ответ (30 секунд)

После построения Render Tree браузер проходит три стадии. **Layout (reflow)** вычисляет геометрию: координаты, размеры — «где что стоит». **Paint** рисует пиксели: фон, текст, тени — «как выглядит». **Composite** собирает слои (layers) в финальный кадр на GPU — «в каком порядке и где». Изменение `width`/`top` требует layout; изменение `color`/`box-shadow` — только paint; `transform`/`opacity` — только compositing. Понимание этого напрямую связано с плавностью анимаций (60/120 fps).

## Подробно

![[схема layout paint composite]]

### Layout

- Зависит от размеров контейнера, `display`, `position`, `flex`/`grid`, шрифтов.
- **Чтение** геометрии (`getBoundingClientRect`, `offsetWidth`, `scrollTop`) может **принудительно** запустить layout, если перед этим были изменения стилей.
- **Layout thrashing:** чередование записи и чтения.

```js
// ❌ O(n) рефлоу
elems.forEach((el) => (el.style.height = el.clientHeight + 10 + "px"));

// ✅ читаем все, затем пишем все
const heights = elems.map((el) => el.clientHeight);
elems.forEach((el, i) => (el.style.height = heights[i] + 10 + "px"));
```

- С `position: absolute` элемент вынимают из потока — реже провоцирует пересчёт соседей.
- `contain: layout` ограничивает область пересчёта (хорошая практика в виджетах).

### Paint

- **Растеризация** элемента в слой: фон, бордеры, текст, тени, `filter`.
- `color`, `background`, `visibility`, `border-radius`, `box-shadow` меняют только paint.
- Многие элементы рисуются в свои слои (промотированные), например `transform`, `opacity`, `position: fixed`, `will-change`.
- **Paint-сложность** зависит от площади и количества слоёв; частая ошибка — огромные `box-shadow` и `filter: blur`.

### Compositing

- Перенос растеризованных слоёв на GPU и сборка финального кадра.
- **`transform`** и **`opacity`** можно менять без layout и paint — композитор просто перерисовывает слои.
- Создание слоёв: `will-change: transform`, `transform: translateZ(0)` (старый хак), анимации.
- `will-change` — не бесплатно: каждый слой = память GPU, и многослойность может сама замедлить.

### Анимация правильно

```css
.card {
  transform: translateX(0);
  transition: transform 200ms ease;
}
.card.open {
  transform: translateX(100px); /* compositing-only */
}
```

```css
/* ❌ layout + paint на каждый кадр */
.card { transition: left 200ms; }

/* ✅ compositor */
.card { transition: transform 200ms; }
```

`left`/`top`/`width`/`height` → layout+paint. `transform`, `opacity` → лишь compositing.

### Как проверить, что происходит

- **DevTools → Performance**: Main thread → `Layout`, `Update Layer Tree`, `Paint`, `Composite Layers`.
- **Rendering panel → Paint flashing** — подсвечивает перерисовку.
- **Layer borders** — границы слоёв.
- **Rendering → Layout Shift Regions** — подсвечивает сдвиги (CLS).
- `performance.getEntriesByType("paint")` — `first-paint`, `first-contentful-paint`.

### Связь с GC и Event Loop

Долгий JS-таск задерживает кадр (`INP` растёт). 60 fps ⇒ бюджет ~16,7 мс на **всё**: JS + style + layout + paint + composite. 120 Гц — 8,3 мс.

## Связанные вопросы

- [[Как происходит рендер после получения HTML]]
- [[Оптимизация приложения и метрики]]
- [[requestAnimationFrame и requestIdleCallback]]
- [[Event Loop]]

## Ресурсы

- web.dev «Rendering performance»: https://web.dev/articles/rendering-performance
- web.dev «Stick to compositor-only properties»: https://web.dev/articles/stick-to-compositor-only-properties-and-manage-layer-count
