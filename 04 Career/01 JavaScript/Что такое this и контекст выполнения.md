---
type: careers/interview
section: JavaScript
topic: this · контекст
status: ready
created: 2026-11-16
tags: [interview, javascript, this, exec-context]
---

# Контекст выполнения и this

## Краткий ответ (30 секунд)

**Контекст выполнения (execution context)** — служебная структура, в которой движок выполняет код: у неё есть лексическое окружение (переменные, области видимости), `this` и, для функций, информация о возврате. Глобальный контекст, контекст функции и контекст модуля — три вида. **`this`** — не переменная, а ссылка, которую движок определяет в момент вызова функции: для обычного вызова это `undefined` в strict-режиме/`globalThis` в non-strict, для метода — объект слева от точки, для `new` — создаваемый экземпляр, для стрелки — `this` из места создания (лексический), а `call`/`apply`/`bind` задают явно.

## Подробно

### Что такое контекст выполнения

Каждому запуску кода соответствует *execution context*: при входе в функцию движок создаёт окружение (lexical environment), куда попадают параметры, локальные переменные, `this` и ссылка на внешнее окружение (scope chain). Это то, что делает замыкания возможными: переменные живут, пока на окружение есть ссылки.

Порядок создания контекста функции:
1. создаётся environment record;
2. объявляются параметры;
3. обрабатывается hoisting (`var` → `undefined`, `let`/`const` → TDZ до инициализации);
4. определяется `this` (или берётся лексический для стрелок);
5. тело выполняется.

Стек вызовов (call stack) — стек этих контекстов. «Стек переполнен» (RangeError) — рекурсия без базового случая.

### Правила `this`

| Вызов | Кто `this` |
|---|---|
| `fn()` в strict-режиме | `undefined` |
| `fn()` в sloppy-режиме | `globalThis` |
| `obj.fn()` | `obj` (объект перед точкой) |
| `new Fn()` | новый объект с `Fn.prototype` |
| Стрелка `() => {}` | из лексического окружения создания |
| `fn.call(ctx)`, `fn.apply(ctx, args)` | `ctx` |
| `fn.bind(ctx)` | `ctx` навсегда |
| Обработчик `btn.onclick = fn` | `btn` (DOM-специфика) |

```js
const user = {
  name: "Аня",
  greet() { return this.name; },
  greetArrow: () => this?.name,
};

const g = user.greet;
user.greet();   // "Аня"
g();            // undefined / TypeError в strict — потеряли контекст
```

### Потеря контекста — классика

```js
class Timer {
  constructor() {
    this.time = 0;
    setInterval(this.tick, 1000); // this потерян
  }
  tick() { this.time++; } // this === undefined (strict) или window
}
```

Решения: `setInterval(() => this.tick(), 1000)`, `setInterval(this.tick.bind(this), 1000)` или стрелочное поле класса `tick = () => { ... }`.

### Стрелочные функции

У стрелок **нет своего `this`, нет `arguments`, их нельзя вызвать через `new`**. `call`/`apply`/`bind` не подменяют их `this`. Это делает их удобными для колбэков в методах и опасными, если нужен динамический контекст.

### `this` в модулях и Node

- ESM-модуль — top-level `this === undefined`.
- CommonJS — top-level `this === module.exports`.
- В браузере верхний уровень — `window`.
- В строгом режиме (ESM, классы, `"use strict"`) `this` в обычной функции при вызове без объекта — `undefined`.

> [!tip] Формулировка для собеседования
> «Контекст выполнения — это не `this`, а окружение исполнения. `this` — часть этого окружения и определяется точкой вызова, а не определением. У стрелок — исключение: лексический `this`».

## Связанные вопросы

- [[Способы привязать контекст]] — `call`/`apply`/`bind`
- [[Замыкания]]
- [[Виды функций]]

## Ресурсы

- MDN `this`: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/this
- Спецификация: Function Environment Records
