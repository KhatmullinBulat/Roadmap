---
type: careers/interview
section: TypeScript
topic: Типы
status: ready
created: 2026-11-16
tags: [interview, typescript, generics, narrowing]
---

# extends в TypeScript

## Краткий ответ (30 секунд)

`extends` значит **разное в зависимости от контекста**. 1) В `interface A extends B` / `class A extends B` — **наследование** (расширение контракта, в классах ещё и код). 2) В дженериках `<T extends Shape>` — **ограничение** (constraint): какие типы допустимы. 3) В **условных типах** `T extends U ? A : B` — **проверка совместимости** (assignability), то есть сужение/фильтрация, а не наследование. Понимание этих трёх ролей — суть вопроса.

## Подробно

### 1. Наследование (интерфейсы и классы)

```ts
interface Animal { name: string }
interface Dog extends Animal { breed: string }

class Base { log() { console.log("base"); } }
class Child extends Base { log() { super.log(); console.log("child"); } }
```

- `interface extends` — добавляет члены, не копирует рантайм (стирается).
- `class extends` — цепочка прототипов, `super`, реальный код.
- Несколько интерфейсов: `interface A extends B, C`.
- Тип-алиас тоже наследует: `type A = B & { extra: string }`.

### 2. Ограничение дженерика (constraint)

```ts
function largest<T extends { value: number }>(items: T[]): T {
  return items.reduce((a, b) => (a.value >= b.value ? a : b));
}
```

- **Сужает** множество допустимых `T`.
- Внутри доступны члены ограничения (`t.value`).
- Формально проверяется структурная совместимость, а не «наследник».

```ts
// ключи
function get<T, K extends keyof T>(obj: T, key: K): T[K] { ... }
```

### 3. Условный тип — проверка/сужение

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<"hi">;   // true
type B = IsString<number>;  // false
```

- Здесь `extends` — **assignable?** («`T` присваивается в `U`?»), не «наследует».
- Распределяется по union при «голом» параметре:

```ts
type ToArray<T> = T extends any ? T[] : never;
type X = ToArray<string | number>;
// string[] | number[]   (распределение!)
```

Обойти — `[T] extends [any] ? ...`.

- С `infer` извлекает типы:

```ts
type ElementOf<T> = T extends (infer E)[] ? E : never;
type E = ElementOf<string[]>; // string
```

### Сравнение ролей

| Контекст | `extends` | Результат |
|---|---|---|
| `interface A extends B` | наследование | расширенный интерфейс |
| `class A extends B` | наследование | цепочка прототипов + `super` |
| `<T extends U>` | constraint | допустимы только совместимые `U` |
| `T extends U ? X : Y` | assignability | выбор ветки типа |
| `infer` внутри | извлечение | новый параметр |

### Так ли это «наследование»?

Строго — **структурная типизация**: TS проверяет «можно ли присвоить», а не «есть ли в цепочке прототипов». `class Cat extends Animal` — настоящая цепочка; `interface Cat extends Animal` — просто набор членов. Поэтому в условных типах `extends` — это проверка структуры.

### Проблема `enum`-совместимости

```ts
type WithName = { name: string };
type A = WithName extends { name: string; age?: number } ? 1 : 0; // 1
```

`age?` не обязателен, поэтому ограничение выполняется — это частый сюрприз.

### Что отвечать

> «`extends` перегружен. В интерфейсах и классах — наследование. В дженериках — constraint: фильтр допустимых типов. В условных типах — assignability: `T` присваивается к `U`. Строго говоря, в типах TS проверяет не наследование, а структурную совместимость».

## Связанные вопросы

- [[Дженерики]]
- [[keyof и typeof]]
- [[Утилитарные типы]]
- [[interface и type]]

## Ресурсы

- TS handbook Conditional Types: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
