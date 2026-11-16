---
type: careers/interview
section: TypeScript
topic: Типы
status: ready
created: 2026-11-16
tags: [interview, typescript, types]
---

# interface и type

## Краткий ответ (30 секунд)

Оба описывают объектный тип, и в большинстве случаев взаимозаменяемы. Различия: `interface` **открыт для объединения** (declaration merging) и реализуется классами; `type` может быть **объединением, кортежем, примитивом, условным типом, mapped type** — это алиас для любого типа, а не только «формы объекта». `interface` показывает более читаемые ошибки и чуть лучше работает с производительностью компилятора на больших глубинах. Практическое правило: **объектные контракты (и особенно публичный API) — `interface`; всё сложное/функциональное — `type`**.

## Подробно

### Что умеет каждый

| Возможность | `interface` | `type` |
|---|---|---|
| Описать объект | ✅ | ✅ |
| Расширение | `extends` | `&` |
| Объединение `A \| B` | ❌ | ✅ |
| Пересечение | через `extends` | `&` |
| Мэппинг/mapped types | ❌ | ✅ |
| Кортеж | нет | `[a: string, b: number]` |
| Функция | можно, но неуклюже | `type Fn = (a: string) => void` |
| Условные типы | ❌ | ✅ |
| `typeof` | ❌ | ✅ |
| Поддержка `implements` класса | ✅ | ✅ |
| Declaration merging | ✅ | ❌ |
| Рекурсивность | ✅ | ✅ (с оговорками) |

### Declaration merging

```ts
interface Window {
  myLib: { init(): void };
}

interface Window {
  myLibVersion: string;
}

// Оба свойства есть у Window — интерфейсы «склеиваются»
```

Это используют `@types/*`, расширение `Express.Request`, `Window`. `type` так не умеет:

```ts
type Window = { a: string };
type Window = { b: string }; // ❌ Duplicate identifier
```

### Расширение

```ts
interface Animal { name: string }
interface Dog extends Animal { breed: string }

type AnimalT = { name: string };
type DogT = AnimalT & { breed: string };
```

`extends` проверяет совместимость сразу и требует, чтобы родитель был «объектом». `&` не проверяет на этапе объявления и может дать `never` при конфликте:

```ts
type Conflict = { a: string } & { a: number }; // a: never
```

### Производительность и ошибки

- `interface` компилируется в «ссылку» на интерфейс; `type` — в алиас. На практике разница заметна только на очень больших типах.
- Hover и сообщения об ошибках у `interface` обычно читаемее: «Type X is missing property a from type Y».
- `type` из объединения даёт быстрый narrowing.

### Классы

```ts
interface Serializable {
  serialize(): string;
}

type SerializableT = { serialize(): string };

class Doc implements Serializable {} // оба работают
class Doc implements SerializableT {}
```

### Что выбрать (в двух предложениях)

> «Объектные модели и публичные контракты — `interface`: расширяются, реализуются классами, их можно мёржить. Объединения, утилиты, mapped/conditional типы, алиасы примитивов — `type`. Не смешивать ради смешивания».

## Связанные вопросы

- [[Во что компилируются type и interface]]
- [[Дженерики]]
- [[Утилитарные типы]]

## Ресурсы

- TS handbook «Differences»: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces
