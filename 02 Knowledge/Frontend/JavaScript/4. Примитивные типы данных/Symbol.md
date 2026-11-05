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
