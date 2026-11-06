---
type: knowledge
area: Frontend
topic: JavaScript · типы данных
status: draft
created: 2026-11-05
tags: []
---

# Symbol

## Кратко

`symbol` — примитивный тип для гарантированно уникальных значений, в основном для «скрытых» ключей объектов. Каждый `Symbol()` уникален, а `Symbol.for()` даёт общий символа из глобального реестра. Символьные свойства не видны в `Object.keys` и `JSON.stringify`.

## Главное

`symbol` — примитивный тип для уникальных значений. Чаще всего символ используют как ключ свойства объекта, когда нужно избежать конфликта с обычными строковыми ключами.
```js
const id = Symbol("id")
const user = {
  name: "Bulat",
  [id]: 42,
}

console.log(user[id]) // 42
```

Описание в `Symbol("id")` помогает отладке, но не определяет идентичность: каждый вызов `Symbol()` создаёт новое уникальное значение.
```js
Symbol("id") === Symbol("id") // false
```

Для общего символа по строковому ключу существует глобальный реестр символов:
```js
const first = Symbol.for("app.token")
const second = Symbol.for("app.token")
first === second // true
Symbol.keyFor(first) // "app.token"
```

Символьные свойства не попадают в `Object.keys()`, `for...in` и `JSON.stringify()`. Получить собственные символьные ключи можно через `Object.getOwnPropertySymbols()` или `Reflect.ownKeys()`.

В языке есть и общеизвестные символы, например `Symbol.iterator` и `Symbol.toPrimitive`. Они позволяют объектам участвовать в протоколах JavaScript: итерации, преобразовании к примитиву, проверке `instanceof` и других.

`Symbol` вызывается без `new`; `new Symbol()` выбрасывает ошибку.

### Как символы реально используют

**Защита от конфликтов в библиотеках.** Библиотека может писать во внутреннее поле пользовательского объекта, не рискуя перезаписать чужое свойство:

```js
const __internal = Symbol("internal")
obj[__internal] = { cached: true }
```

**Метаданные, не попадающие в сериализацию.** Символьные поля не уйдут в JSON и не попадут в `for...in`, поэтому подходят для пометок.

**Протоколы языка.** `Symbol.iterator` делает объект итерируемым:

```js
class Range {
  constructor(from, to) {
    this.from = from
    this.to = to
  }
  *[Symbol.iterator]() {
    for (let i = this.from; i < this.to; i++) yield i
  }
}

[...new Range(0, 3)] // [0, 1, 2]
```

Другие общеизвестные символы: `Symbol.asyncIterator`, `Symbol.toPrimitive`, `Symbol.hasInstance`, `Symbol.toStringTag`.

### Чего не стоит ожидать

- Символ **не** делает свойство по-настоящему приватным: `Object.getOwnPropertySymbols()`, `Reflect.ownKeys()` и `Object.assign()` его видят.
- Символы нельзя сериализовать, поэтому для DTO и API используют строковые ключи.
- Символ не участвует в строковом ключе: `obj[Symbol("a")]` и `obj.a` — разные свойства.

## Практика

- [ ] Создать объект с символьным ключом и проверить, что `Object.keys` его не показывает
- [ ] Сравнить `Symbol()` и `Symbol.for()` в двух вызовах
- [ ] Реализовать класс с `[Symbol.iterator]` и обойти его через `for...of`

## Связанные темы

- [[Типы данных]]
- [[1. Object | Object]]
- [[2. Оператор typeof | Оператор typeof]]
- [[String]]

## Ресурсы

- Документация: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Symbol
- Статьи: https://doka.guide/js/symbol/ · https://learn.javascript.ru/symbol

