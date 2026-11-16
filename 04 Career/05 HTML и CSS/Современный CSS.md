---
type: careers/interview
section: HTML и CSS
topic: CSS
status: ready
created: 2026-11-16
tags: [interview, css, modern-css]
---

# Современный CSS

## Краткий ответ (30 секунд)

Набор возможностей, которые убрали старые костыли: **CSS-переменные** (темы, токены), **`clamp()`/`min()`/`max()`** (плавные размеры), **`:has()`** (селектор «родитель по содержимому»), **`:is()`/`:where()`**, **nesting** (нативный вложенный CSS), **`@layer`** (каскад), **container queries** (компонентные брейкпоинты), **`@property`** (типизированные переменные), **color-mix/oklch** (цвета), **`aspect-ratio`**, **`gap`**, **`dvh/svh`**, **subgrid**, **scroll-driven animations**. Для собеседования достаточно уверенно владеть переменными, `clamp`, `:has`, `@layer`, container queries и nesting.

## Подробно

### CSS-переменные

```css
:root {
  --color-primary: #2563eb;
  --space-2: 8px;
  --radius: 8px;
}

.button {
  background: var(--color-primary);
  padding: var(--space-2) calc(var(--space-2) * 2);
  border-radius: var(--radius);
}

/* Тёмная тема — переопределение переменных */
[data-theme="dark"] { --color-primary: #60a5fa; }
```

- Наследуются по DOM; не имеют типов (до `@property`).
- Меняются в рантайме: `el.style.setProperty("--x", "10px")`.
- **Не** участвуют в каскаде как свойства — только как значения.

### `clamp()`

```css
.title { font-size: clamp(1.25rem, 1rem + 2vw, 2.5rem); }
.sidebar { width: clamp(200px, 25%, 320px); }
```

- `clamp(min, ideal, max)` = `max(min, min(ideal, max))`.
- Убирает «прыжки» между breakpoints.

### `:has()` (Baseline 2023)

```css
/* Карточка с картинкой — другой отступ */
.card:has(> img) { padding-top: 0; }

/* Форма с невалидным полем */
form:has(input:invalid) { border-color: red; }

/* Сосед по ховеру */
.card:has(+ .card:hover) { ... }
```

Не поддерживается IE и старым Safari (<15.4). Не использовать для критичного layout без фолбэка.

### `:is()` и `:where()`

```css
:where(h1, h2, h3) { margin-block: 0; }  /* нулевая специфичность — легко переопределить */
.card :is(h2, h3) { ... }               /* специфичность как у самого сильного */
```

### Nesting

```css
.card {
  padding: 16px;

  &__title { font-weight: 600; }   /* работает как в SCSS */
  &:hover { background: #f5f5f5; }
  @media (width < 600px) { padding: 8px; }
}
```

- Нативный с 2023; `&` обязателен для «родителя» в начале.
- Вложенность **стала `:is()`-подобной** по специфичности — не злоупотребляйте.

### `@layer`

```css
@layer reset, base, components, utilities;

@layer components {
  .button { background: var(--color-primary); }
}
```

- Управляет каскадом без `!important`.
- Неслоевые стили сильнее слоёв.

### Container queries

```css
.card-wrap { container-type: inline-size; }

@container (width > 400px) {
  .card { flex-direction: row; }
}
```

### Другие «не мелочи»

- **`aspect-ratio: 16 / 9`** — вместо «padding-hack» (и помогает CLS).
- **`gap`** — в flex и grid.
- **`dvh/svh/lvh`** — высота viewport по-настоящему.
- **`@property`** — тип и анимация кастомных свойств.
- **`color-mix(in oklch, ...)`**, `oklch()` — перцептивные цвета и прозрачность.
- **`scroll-driven animations`** (`animation-timeline: view()`) — прогресс при скролле без JS.
- **`text-wrap: balance/pretty`** — красивые заголовки/абзацы.
- **`accent-color`**, `color-scheme` — нативные контролы под тему.

### Чего больше не стоит делать

- `float` для раскладки, `clearfix`.
- Хак `padding-top: 56.25%` для соотношения (есть `aspect-ratio`).
- Множество jQuery-анимаций (есть `transition`/`animation`, `@starting-style`).
- Префиксы вручную (`-webkit-`) — автопрефиксер остался для узких кейсов.

> [!tip] Как отвечать
> «Из современного CSS я чаще всего использую переменные, `clamp`, `:has`, нативный nesting, `@layer` и container queries. Они убирают костыли: JS-подгонку размеров, `!important` и репетиции медиа-запросов в каждом компоненте».

## Связанные вопросы

- [[Флексбокс и грид]]
- [[Адаптивность и медиа-запросы]]
- [[CSS каскад и специфичность]]

## Ресурсы

- Can I use: https://caniuse.com/
- MDN CSS: https://developer.mozilla.org/ru/docs/Web/CSS
