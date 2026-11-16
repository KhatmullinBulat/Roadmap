---
type: careers/interview
section: JavaScript
topic: Коллекции
status: ready
created: 2026-11-16
tags: [interview, javascript, weakref, memory]
---

# WeakMap и WeakSet

## Краткий ответ (30 секунд)

`WeakMap`/`WeakSet` — коллекции со **слабыми** ссылками на ключи (для `WeakSet` — на значения): если на объект-ключ нет других ссылок, запись автоматически удаляется сборщиком мусора. Поэтому они не мешают освобождению памяти. Ограничения: ключи **только объекты** (с ES2023 — ещё и незарегистрированные `Symbol`), **нет итерации**, **нет `size`**, **нет `clear`**. Применяются для кэшей, привязанных к объектам, приватных данных, отслеживания DOM-узлов.

## Подробно

### Сравнение

| | `Map` / `Set` | `WeakMap` / `WeakSet` |
|---|---|---|
| Ключи | любые | только объекты и незарегистрированные `Symbol` |
| Ссылка на ключ | сильная | **слабая** |
| Мешает GC | да (пока коллекция жива) | нет |
| `size` | да | нет |
| Итерация | да | нет |
| `clear()` | да | нет |
| `forEach`/`keys`/`values` | да | нет |
| `has/get/set/delete` | да | да (у `WeakSet` — `add/has/delete`) |

Отсутствие итерации — не случайность: если можно перебрать ключи, значит, их держат достижимыми, а тогда GC не может их освобождать.

### Пример 1: кэш «при объекте»

```js
const cache = new WeakMap();

function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = expensive(obj);
  cache.set(obj, result);
  return result;
}

let user = { id: 1 };
process(user);
process(user); // из кэша

user = null; // когда других ссылок нет — запись в WeakMap тоже исчезнет
```

`Map` в той же роли удерживал бы `user` навсегда (утечка).

### Пример 2: приватность

```js
const privateData = new WeakMap();

class Counter {
  constructor() {
    privateData.set(this, { count: 0 });
  }
  inc() {
    privateData.get(this).count++;
  }
}

const c = new Counter();
c.inc();
c.count; // undefined
```

Сейчас на практике чаще `#count` (нативные приватные поля), которые семантически похожи.

### Пример 3: отметка посещённых

```js
const visited = new WeakSet();

function track(node) {
  if (visited.has(node)) return;
  visited.add(node);
  // обработка
}
```

DOM-узлы, которые удалили из документа и на которые больше нет ссылок, освободятся вместе с отметкой.

### Пример 4: метаданные для объектов

```js
const meta = new WeakMap();
meta.set(element, { tooltip: "..." }); // без мутации DOM и без утечек
```

`dataset`/`expando`-свойства — альтернатива, но они видны снаружи и могут конфликтовать.

### Про символы

С ES2023 ключами `WeakMap`/`WeakSet` разрешены **незарегистрированные** `Symbol` (не из `Symbol.for()`): они уникальны, не пересоздаются и тоже могут быть освобождены. Так можно связать данные с «идентичностью», не удерживая её. Ключ-символ в обычном `Map` этого не даёт.

### После освобождения

```js
const ref = new WeakRef(target); // ref.deref() → target | undefined
```

`WeakRef` и `FinalizationRegistry` — низкоуровневые инструменты. Не путайте их с `WeakMap`: `WeakRef` позволяет **проверить**, жив ли объект, но поведение может меняться от GC.

> [!tip] Как отвечать
> «WeakMap/WeakSet — слабые ссылки: запись исчезает сама, когда объект больше не достижим. Поэтому нет итерации и `size` — иначе GC не смог бы освобождать. Используем для кэшей и метаданных, привязанных к объектам».

## Связанные вопросы

- [[Map и объект]]
- [[Сборщик мусора и достижимость]]

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
