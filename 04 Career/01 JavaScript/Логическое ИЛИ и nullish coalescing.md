---
type: careers/interview
section: JavaScript
topic: Операторы
status: ready
created: 2026-11-16
tags: [interview, javascript, operators]
---

# Оператор || vs ??

## Краткий ответ (30 секунд)

`||` возвращает правый операнд, если левый **ложный** (falsy): `0`, `""`, `false`, `null`, `undefined`, `NaN`. `??` возвращает правый операнд, только если левый **`null` или `undefined`**. Поэтому `0 || 5` даёт `5`, а `0 ?? 5` — `0`. `??` используют для значений по умолчанию, когда `0`, `""` и `false` — валидные данные.

## Подробно

### Таблица

| Выражение | Результат | Почему |
|---|---|---|
| `0 \|\| 10` | `10` | `0` — falsy |
| `0 ?? 10` | `0` | `0` — не nullish |
| `"" ?? "default"` | `""` | |
| `false ?? true` | `false` | |
| `null \|\| "x"` | `"x"` | |
| `undefined ?? "x"` | `"x"` | |

### Ключевые слова

- **Logical OR (`||`)** — оператор логических значений, работает через `ToBoolean`.
- **Nullish coalescing (`??`, ES2020)** — «объединение по nullish», проверяет только `null`/`undefined`.
- **Falsy-значения:** `false, 0, -0, 0n, "", null, undefined, NaN`.
- **Nullish-значения:** только `null` и `undefined`.

### Смешивание с && / ||

Нельзя писать `a ?? b || c` без скобок — `SyntaxError`. Правило: `??` нельзя смешивать с `||`/`&&` без явных скобок.

```js
const value = a ?? (b || c); // OK
// const value = a ?? b || c; // SyntaxError
```

### Частые практические ситуации

```js
// Опции компонента: 0 — валидное значение
function render({ width }) {
  const w = width ?? 320; // width: 0 не заменится на 320
}

// Ответ API
const count = response.count ?? 0; // 0 останется 0
```

### Родственные операторы

- `?.` (optional chaining): `user?.address?.city ?? "—"`.
- `??=` (логическое присваивание по nullish): `opts.theme ??= "light"` — задать, только если сейчас `null`/`undefined`.
- `||=` — присвоить, если falsy; `&&=` — присвоить, если truthy/есть (логическое И с присваиванием).

> [!warning] Для собеседования
> Вопрос почти всегда проверяет понимание `0`/`""`/`false`. Добавьте фразу «`??` ввели для того, чтобы не терять `0` и пустую строку в значениях по умолчанию».

## Связанные вопросы

- [[any, unknown и never]]
- [[Promise и async await]] — `Promise.resolve(v ?? 0)`

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing
