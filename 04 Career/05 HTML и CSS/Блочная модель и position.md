---
type: careers/interview
section: HTML и CSS
topic: Раскладка
status: ready
created: 2026-11-16
tags: [interview, css, box-model, position]
---

# Блочная модель и position

## Краткий ответ (30 секунд)

**Блочная модель** — каждый элемент это прямоугольник: `content → padding → border → margin`. По умолчанию `box-sizing: content-box` (ширина = контент), с `border-box` ширина включает padding и border — поэтому его ставят глобально (`*, *::before, *::after { box-sizing: border-box; }`). `margin` складывается у соседей (**margin collapsing**), `padding` — нет. **`position`** — `static` (по умолчанию), `relative` (сдвиг относительно себя, остаётся в потоке), `absolute` (от ближайшего позиционированного предка, вынимается), `fixed` (от viewport, при прокрутке неподвижен), `sticky` (гибрид, «прилипает» в пределах scroll-контейнера).

## Подробно

### Блочная модель

```
+--------------------------- margin
|  +--------------------- border
|  |  +--------------- padding
|  |  |  +--------- content
|  |  |  |
```

```css
/* Разные ширины при width: 200px */
.content-box { box-sizing: content-box; padding: 20px; border: 2px; } /* 244px */
.border-box  { box-sizing: border-box;  padding: 20px; border: 2px; } /* 200px */
```

- `box-sizing: border-box` используют почти всегда — легче считать.
- `margin: 0 auto` центрирует блочный элемент с `width`.
- **`margin collapsing`**: вертикальные margin двух соседей не складываются, а «схлопываются» до большего; не происходит при `padding`/`border`/`display: flex|grid`, для `float`, `absolute`.
- `width: 100%` + `padding` при `content-box` даёт переполнение.

### `position`

| Значение | Поток | Отсчёт | `z-index` |
|---|---|---|---|
| `static` | да | — | z-index не работает |
| `relative` | да | от себя | работает |
| `absolute` | вынуте | ближайший предок с `position != static` | работает |
| `fixed` | вынуте | viewport | работает |
| `sticky` | да | прилипает к порогу `top/left...` | работает |

```css
.modal {
  position: fixed;
  inset: 0;                /* то же, что top: 0; right: 0; bottom: 0; left: 0 */
  display: grid;
  place-items: center;
  z-index: 100;
}
```

- **`inset`** — шорткат для `top/right/bottom/left` (Baseline 2021).
- `absolute` внутри `relative` — классический паттерн «бейдж в углу карточки».
- `fixed` может «ломаться» из-за предка с `transform`, `filter`, `perspective`, `will-change`, `contain` — он создаёт **containing block** для fixed.
- `sticky` не работает, если у предка `overflow: hidden/auto/scroll` по нужной оси или высоты контейнера не меньше элемента.

### `z-index`

- Работает только у позиционированных элементов (`relative`, `absolute`, `fixed`, `sticky`) и у flex/grid-детей.
- Порядок наложения: фон → border → потомки с отрицательным z-index → текст/содержимое → позиционированные (`z-index`).
- **Stacking context** создают: `position + z-index`, `opacity < 1`, `transform`, `filter`, `isolation: isolate`, `will-change`, `contain: paint`.
- Поэтому `z-index: 9999` не поможет, если родитель создал свой stacking context — «выбраться» из него нельзя.

### Float

Остаётся для обтекания текста вокруг картинки; для раскладки не используется (flex/grid). `clear` — отмена обтекания.

### Что спросят

- «Почему `100%` + `padding` вылезает за край?» → `content-box`, нужен `border-box`.
- «Почему sticky не работает?» → `overflow` у родителя или высота контейнера.
- «Почему `z-index` не помогает?» → родитель создаёт stacking context.
- «Как центрировать?» → `grid`/`place-items`, `flex`+`margin:auto`, `absolute`+`inset:0`+`margin:auto`.

> [!warning] Частая ошибка
> Менять `position: absolute` без `relative` родителя — элемент отсчитывается от `<body>`/ближайшего позиционированного предка. На собеседовании это типичный разбор «почему элемент убежал».

## Связанные вопросы

- [[Флексбокс и грид]]
- [[CSS каскад и специфичность]]
- [[Современный CSS]]

## Ресурсы

- MDN Box model: https://developer.mozilla.org/ru/docs/Web/CSS/CSS_box_model
- MDN `position`: https://developer.mozilla.org/ru/docs/Web/CSS/position
