---
type: knowledge
area: Frontend
topic: JavaScript · типы данных
status: draft
created: 2026-11-05
tags: []
---

# Boolean

## Кратко

`boolean` — логический тип с двумя значениями, `true` и `false`. В условиях любое значение неявно приводится к нему: ложными считаются только `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined` и `NaN`, всё остальное — истинно.

## Главное

`boolean` — примитивный тип с двумя значениями: `true` и `false`. Его используют для состояний и результатов проверок.
```js
const isLoggedIn = true
const hasAccess = user.role === "admin"
```

В условии JavaScript неявно преобразует любое значение к логическому. Ложными (*falsy*) считаются только:
```js
false, 0, -0, 0n, "", null, undefined, NaN
```

Все остальные значения истинны (*truthy*), в том числе пустые массивы, пустые объекты и строка `"false"`.
```js
Boolean([]) // true
Boolean("false") // true
```

Явно преобразовать значение можно через `Boolean(value)` или двойное отрицание `!!value`. Второй вариант короче, но первый обычно легче читать.
```js
const hasName = Boolean(user.name)
```

Не используйте `new Boolean(false)`. Это объект, а любой объект является truthy, поэтому `if (new Boolean(false))` выполнится.

### Truthy/falsy подвохи

```js
Boolean([]) // true
Boolean({}) // true
Boolean("0") // true — непустая строка
Boolean(" ") // true — пробел это символ

Boolean(NaN) // false
Boolean(-0) // false
```

Частая ошибка — проверять массив через `if (arr)` и считать, что пустой массив «ложный». Для массивов и строк проверяйте длину явно: `arr.length > 0`.

```js
const items = []
if (items) console.log("выполнится") // массив всегда truthy
if (items.length) console.log("не выполнится")
```

### Короткое замыкание и `??`

`&&` и `||` возвращают не `boolean`, а одно из исходных значений:
```js
0 || "значение" // "значение"
false || 5 // 5
"" && "x" // ""
null ?? "default" // "default"
0 ?? "default" // 0 — ?? не считает 0 ложным
```

`||` срабатывает на любое ложное значение (включая `0` и `""`), а `??` — только на `null` и `undefined`. Для свойств и параметров это важное различие.

## Практика

- [ ] Построить таблицу всех falsy-значений и заполнить её примерами
- [ ] Показать разницу `count || 10` и `count ?? 10` при `count = 0`
- [ ] Найти в проекте проверки вида `if (array)` и заменить на `.length`

## Связанные темы

- [[Undefined]]
- [[Null]]
- [[Типы данных]]
- [[1. Преобразование типов | Преобразование типов]]

## Ресурсы

- Документация: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Boolean
- Статьи: https://doka.guide/js/boolean/ · https://learn.javascript.ru/ifelse

