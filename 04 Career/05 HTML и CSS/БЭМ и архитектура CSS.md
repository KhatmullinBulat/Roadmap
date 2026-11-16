---
type: careers/interview
section: HTML и CSS
topic: Архитектура CSS
status: ready
created: 2026-11-16
tags: [interview, css, bem, architecture]
---

# БЭМ и методы организации CSS

## Краткий ответ (30 секунд)

**БЭМ** (Блок, Элемент, Модификатор) — соглашение об именовании классов: блок `.card`, элементы `.card__title`, модификаторы `.card--highlighted`. Цель — **плоская, предсказуемая специфичность** (не вложенные селекторы), независимость блоков (можно переносить) и поиск класса по названию. Альтернативы: **CSS-модули** (локальные имена на компонент), **CSS-in-JS** (styled-components/emotion), **utility-first** (Tailwind), **ITCSS/CUBE/BEMIT** для структуры, **`:where()`/`@layer`** для контроля каскада.

## Подробно

### БЭМ

```html
<div class="card card--highlighted">
  <h2 class="card__title">Товар</h2>
  <button class="card__button button button--primary">Купить</button>
</div>
```

- **Блок** — самостоятельный компонент.
- **Элемент** — часть блока (не планируем отдельно).
- **Модификатор** — вариация (`--size-s`, `--active`, `--theme-dark`).
- **Микс** — несколько классов у одного элемента (позиция + компонент).

```css
.card { padding: 16px; }
.card__title { margin: 0 0 8px; }
.card--highlighted { border-color: gold; }
.button--primary { background: blue; }
```

Правила: **один класс — одна роль**, не вкладывать селекторы (`.card .title`), не использовать id, не модификатор для модификатора (`.card--highlighted__title` — запрещено).

**Плюсы:** предсказуемость, нулевая специфичность (одни классы), переиспользование, понятная структура.
**Минусы:** длинные имена, риск «класса-хаоса» без лентера (`stylelint`), смешивание ответственности при неудачном разбиении.

### CSS-модули

```css
/* Button.module.css */
.button { background: blue; }
```

```js
import s from "./Button.module.css";
<button className={s.button}>...</button>
```

- Имена хешируются (`Button_button__aB3x1`) — локальные, не конфликтуют.
- Сейчас стандарт в Vite/Next/CRA; идеален для компонентов.
- Минус: динамические классы через `s[...]`, «плывущее» чтение HTML.

### CSS-in-JS

- `styled-components`, `emotion`, `vanilla-extract` (zero-runtime).
- Плюсы: стили рядом с логикой, темы, динамика.
- Минусы: рантайм-стоимость (для styled-components), больше JS, серверный рендеринг сложнее. Trend — уход в CSS-модули/zero-runtime (vanilla-extract, Linaria, Panda CSS).

### Utility-first (Tailwind)

```html
<button class="rounded bg-blue-600 px-4 py-2 text-white hover:bg-blue-700">
  Купить
</button>
```

- Плюсы: быстро, размер CSS ограничен (только используемые утилиты), нет роста зоопарка.
- Минусы: длинный `class`, стили в разметке, нужно знать токены; для сложных компонентов — `@apply`/вложенные плагины.

### ITCSS / CUBE / слои

- **ITCSS:** структура по специфичности — settings → tools → generic → elements → objects → components → utilities.
- **CUBE CSS:** compositional (классы-утилиты + блоки), exception.
- **`@layer`** — современная замена «ITCSS-слоёв»: `@layer reset, base, components, utilities`.

### Вопрос «как вы организуете CSS»

> «Компонентные стили через CSS-модули или Tailwind. БЭМ — для именования классов в обычном CSS/Sass. Специфичность держим плоской: один класс на элемент, `@layer` и `:where()` для базовых стилей. Глобальные токены (цвета/размеры) — CSS-переменные».

### Подвох

«БЭМ — это про плоский CSS или про JavaScript?» — про **соглашение**, оно применимо и к CSS-модулям (`.card__title`), и к разметке. И не путайте «модификатор» с «элементом»: `.card__title--big` — модификатор элемента, допустим, а `.card--big__title` — нет.

## Связанные вопросы

- [[CSS каскад и специфичность]]
- [[Современный CSS]]
- [[Флексбокс и грид]]

## Ресурсы

- БЭМ: https://ru.bem.info/methodology/
- Tailwind: https://tailwindcss.com/
