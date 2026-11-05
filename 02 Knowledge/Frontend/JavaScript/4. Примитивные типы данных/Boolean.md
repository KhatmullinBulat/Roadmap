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
