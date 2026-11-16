---
type: careers/interview
section: Браузер и сети
topic: Рендеринг
status: ready
created: 2026-11-16
tags: [interview, browser, rendering]
---

# Как происходит рендер после получения HTML

## Краткий ответ (30 секунд)

Последовательность: **парсинг HTML → DOM**, **парсинг CSS → CSSOM**, затем **Render Tree** (видимые узлы DOM + вычисленные стили), **Layout** (геометрия и позиции), **Paint** (растеризация слоёв в пиксели), **Compositing** (слои собираются GPU и выводятся на экран). JS может влиять на каждый шаг: блокировать парсер (`script` без `async`/`defer`), вызывать дорасчёт стилей и layout (`reflow`), инициировать repaint. Ключ к производительности — не дёргать layout в цикле, менять `transform`/`opacity` (compositor-only), выносить тяжёлый JS/анимации.

## Подробно

### 1. Парсинг HTML

- **Токенизатор → дерево узлов**. Парсер инкрементальный: HTML можно отдавать по частям (streaming).
- **Preload scanner** параллельно находит ресурсы (`img`, `link`, `script`) и запускает их загрузку, не дожидаясь парсинга.
- `<script>` **без** `async`/`defer` **блокирует парсинг**: нужно выполнить скрипт, потому что он может вызвать `document.write` или изменить DOM.
- `<script defer>` выполняется после парсинга, в порядке документов; `<script async>` — как только загрузится; `<script type="module">` — по умолчанию `defer`.
- `<link rel="stylesheet">` **блокирует рендер**, но не парсинг — страница может «собираться» целиком.

### 2. DOM

- Итог — **Document Object Model**: дерево узлов, доступное JS.
- Внешние изменения (`document.getElementById(...).remove()`) сразу меняют DOM, но не экран.

### 3. CSSOM

- CSS-файлы и `<style>` парсятся в **CSSOM** (CSS Object Model).
- Каскад, специфичность, наследование вычисляются в **computed styles**.
- CSSOM нужен для построения Render Tree, поэтому внешние стили блокируют первую отрисовку.

### 4. Render Tree (или Layout Tree)

- Комбинируются DOM + CSSOM: узлы, которые **видны** (`display: none` выкидывается, `visibility: hidden` остаётся в дереве, хотя не рисуется).
- `::before`/`::after` добавлены в дерево.

### 5. Layout (Reflow)

- **Геометрия**: ширина, высота, координаты элементов.
- Зависит от `width`, шрифтов, `position`, flex/grid, `float`.
- Вызов `offsetHeight`, `getBoundingClientRect`, `scrollTop` **синхронно** заставляет браузер сделать layout (читается «живая» геометрия).
- **Layout Thrashing** — чередование чтения геометрии и записи стилей в цикле:

```js
// ❌ каждый шаг: чтение → запись → чтение → запись
for (const el of items) {
  el.style.width = el.offsetWidth + 10 + "px";
}

// ✅ сначала прочитать, потом записать
const widths = items.map((el) => el.offsetWidth);
items.forEach((el, i) => (el.style.width = widths[i] + 10 + "px"));
```

### 6. Paint

- **Растеризация**: элементы рисуются в слои (paint layers) на CPU/GPU.
- Сложность зависит от количества слоёв, `box-shadow`, `filter`, `border-radius`.

### 7. Compositing

- Слои собираются в кадр композутором (GPU).
- **`transform`/`opacity`** могут анимироваться на композиторе без пересчёта layout/paint → самая дешёвая анимация.
- `will-change: transform` создаёт слой заранее (но злоупотребление = много памяти).
- `position: fixed`/`sticky` и `filter` тоже могут создавать слои.

### Что вызывает что

| Действие | Пересчёт |
|---|---|
| `color`, `background-color` | paint → compositing |
| `width`, `padding`, `font-size` | layout → paint → compositing |
| `transform`, `opacity` | compositing |
| `display: none` | layout (удалить из дерева) |
| Добавление DOM | layout |

### Ресурсы, которые нужно ждать

- **CSS** — блокирует рендер.
- **Критичные шрифты** — `@font-face` с `font-display: swap/optional`; без этого может «мигать» текст (FOIT/FOUT → CLS).
- **JS** — если блокирует парсер, всё остальное ждёт.
- **Картинки** — не блокируют DOM, но занимают место; без `width`/`height` → CLS.

### Влияние на метрики

- **FCP** — первая отрисовка (нельзя раньше, чем загрузятся CSS и критичные шрифты).
- **LCP** — самый крупный элемент обычно картинка/заголовок; ускоряйте приоритетными preload.
- **CLS** — layout-сдвиги от поздних изображений, шрифтов, баннеров.

## Связанные вопросы

- [[Рендер layout paint compositing]]
- [[Оптимизация приложения и метрики]]
- [[Что такое Shadow DOM]]
- [[Что происходит при вводе URL]]

## Ресурсы

- web.dev Rendering: https://web.dev/articles/rendering-performance
- Chrome DevTools: https://developer.chrome.com/docs/devtools/performance
