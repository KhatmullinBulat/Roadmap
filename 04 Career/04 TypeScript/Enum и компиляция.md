---
type: careers/interview
section: TypeScript
topic: Компиляция
status: ready
created: 2026-11-16
tags: [interview, typescript, compilation, enum]
---

# Enum и во что он компилируется

## Краткий ответ (30 секунд)

`enum` — единственная конструкция TypeScript, которая порождает **JS-код**: обычный enum компилируется в **IIFE и объект с обратным маппингом** (`Enum[0] = "A"`). Поэтому enum **существует в рантайме** и попадает в бандл. `const enum` компилируется в **инлайн-значения** (без объекта), но требует `isolatedModules`-совместимости и ломает `preserveConstEnums`. `string`-enum компилируется в объект без обратного маппинга. Современная альтернатива — **union литералов** + `as const` объект: нет рантайм-кода, лучше tree-shaking и совместимость.

## Подробно

### Обычный enum

```ts
enum Color {
  Red,   // 0
  Green, // 1
  Blue,  // 2
}
```

Компилируется (упрощённо) в:

```js
var Color;
(function (Color) {
  Color[(Color["Red"] = 0)] = "Red";
  Color[(Color["Green"] = 1)] = "Green";
  Color[(Color["Blue"] = 2)] = "Blue";
})(Color || (Color = {}));
```

Итог — объект:

```js
Color.Red;   // 0
Color[0];    // "Red"  ← обратный маппинг
Object.keys(Color); // ["0","1","2","Red","Green","Blue"]
```

- **IIFE** нужна из-за того, что enum — переменная и объект одновременно.
- **`Object.values(Color)`** даст и числа, и строки — частая ошибка при итерации.

### String enum

```ts
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
}
```

```js
var Status;
(function (Status) {
  Status["Active"] = "ACTIVE";
  Status["Inactive"] = "INACTIVE";
})(Status || (Status = {}));
```

- Обратного маппинга **нет** (строки не могут быть ключами так же).
- Рантайм-объект всё равно есть.

### `const enum`

```ts
const enum Direction { Up, Down }
const d = Direction.Up;
```

Компилируется в `var d = 0;` — **без объекта**. Плюс: меньше кода. Минусы:

- **`isolatedModules: true`** (Vite/esbuild/SWC, Babel) не может «докомпилировать» значения — предупреждение/ошибка «Cannot access ambient const enums when 'isolatedModules' is enabled».
- `preserveConstEnums: true` сохраняет объект — тогда уже не «const».
- Работа с динамическими значениями невозможна: `Direction[x]` нельзя.
- Экспорт из библиотеки и импорт в другой пакет — значение может быть вырезано; работает только если `const enum` в проекте с полным `tsc`.

### Амбиентные

```ts
declare enum External { A, B }
```

Компилируется в **ничего**: вы утверждаете, что объект где-то существует (например, из глобального окружения).

### Проблемы enum

- Рантайм-код и вес бандла (всё, что не деревошейкится — объект).
- Дублирование значений между пакетами.
- Числовой enum проверяет совместимость слабо: `const c: Color = 42` — ошибка, но `const c: Color = 1 as Color` легко подделать, и любое число проходит при `number`.
- Обратный маппинг сбивает итерацию.
- В `isolatedModules`/`verbatimModuleSyntax` — дополнительная возня.

### Современная альтернатива

```ts
// 1. Union литералов
type Status = "active" | "inactive";

// 2. as const-объект + typeof
const Status = {
  Active: "ACTIVE",
  Inactive: "INACTIVE",
} as const;

type Status = (typeof Status)[keyof typeof Status]; // "ACTIVE" | "INACTIVE"

// 3. Использование
const s: Status = Status.Active; // автодополнение + тип
```

Плюсы: нет рантайм-магии (кроме самого объекта), tree-shaking, значения сами собой JSON-совместимы, проще с бэком.

### Что отвечать

> «enum — единственное, что TypeScript генерирует. Обычный — IIFE и объект с обратным маппингом, const — инлайн. Из-за рантайма и isolatedModules я предпочитаю union литералов или `as const`-объект, особенно в библиотеках».

## Связанные вопросы

- [[Во что компилируются type и interface]]
- [[any, unknown и never]]
- [[Утилитарные типы]]

## Ресурсы

- TS handbook Enums: https://www.typescriptlang.org/docs/handbook/enums.html
