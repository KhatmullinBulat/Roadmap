---
type: careers/interview
section: TypeScript
topic: Типы
status: ready
created: 2026-11-16
tags: [interview, typescript, keyof, typeof]
---

# keyof и typeof

## Краткий ответ (30 секунд)

`keyof T` — **оператор типа**, возвращает объединение ключей (строк/символов/чисел): `keyof { a: 1; b: 2 }` = `"a" | "b"`. Работает с любым типом (не только объектом). `typeof value` — **оператор значения**, но в позиции типа возвращает **тип** переменной/объекта: `const x = { a: 1 }; type T = typeof x;` → `{ a: number }`. Вместе: `type Keys = keyof typeof config` — «ключи типа объекта config». Часто путают с JS `typeof` (строка `"number"`), который вернёт значение.

## Подробно

### `keyof`

```ts
interface User {
  id: number;
  name: string;
  42: boolean;
}

type UserKeys = keyof User; // "id" | "name" | 42
```

- Возвращает **union литералов** ключей.
- Применим к union: `keyof (A | B)` — **пересечение ключей**; `keyof (A & B)` — **объединение**.
- `keyof any` → `string | number | symbol`.
- Для индексируемого типа:

```ts
interface Dict { [key: string]: number }
type K = keyof Dict; // string | number
```

- Часто используется:

```ts
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

type Keys = keyof typeof config;
```

### `typeof` в типах

```ts
const config = {
  api: "https://example.com",
  retries: 3,
} as const;

type Config = typeof config;
// { readonly api: "https://example.com"; readonly retries: 3 }

type ApiUrl = (typeof config)["api"]; // "https://example.com"
```

- Это **не** JS-`typeof value === "object"`. В позиции типа `typeof x` возвращает тип значения `x`.
- `typeof` работает только с **значениями в области видимости** (переменные, функции, импорты).
- С `as const` значения становятся литералами (полезно для констант).
- `typeof import("./module")` — тип модуля без импорта значений (для **типов**).

### Комбинация

```ts
const ROUTES = {
  home: "/",
  profile: "/profile",
  settings: "/settings",
} as const;

type RouteName = keyof typeof ROUTES;       // "home" | "profile" | "settings"
type RoutePath = (typeof ROUTES)[RouteName]; // "/" | "/profile" | "/settings"

function navigate(path: RoutePath) {}
navigate("/profile"); // ✅
navigate("/no");      // ❌
```

Это самый частый практический запрос в реальном коде.

### `keyof` в дженериках

```ts
function pluck<T, K extends keyof T>(items: T[], key: K): T[K][] {
  return items.map((item) => item[key]);
}

const names = pluck(users, "name"); // string[]
```

`T[K]` — **indexed access type**: тип значения по ключу.

### Отличие от `keyof typeof` в классах

```ts
class A { x = 1; y() {} }
const a = new A();

type InstanceKeys = keyof A;          // "x" | "y"
type StaticKeys = keyof typeof A;      // ключи конструктора
type ValueKeys = keyof typeof a;       // "x" | "y"
```

`A` в типовой позиции — **тип экземпляра**; `typeof A` — тип **конструктора** (статики).

### Частые ошибки

- `type T = keyof "abc"` → даёт методы `string` (`length`, `charAt`...) — `keyof` на примитиве даёт ключи его обёртки.
- `typeof User` (где `User` — тип) — ошибка: `typeof` ожидает значение, а не тип.
- `keyof` объекта-значения, а не типа — `keyof {a:1}` допустим (литеральный тип).

> [!tip] Как отвечать
> «`keyof` — тип, объединение ключей. `typeof` — берёт тип значения. Вместе `keyof typeof obj` дают литеральные ключи объекта, что даёт автодополнение и проверку. С `as const` это стандартный способ типизировать константы-мапы».

## Связанные вопросы

- [[Дженерики]]
- [[Утилитарные типы]]
- [[any, unknown и never]]

## Ресурсы

- TS handbook `keyof`: https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
- TS handbook `typeof`: https://www.typescriptlang.org/docs/handbook/2/typeof-types.html
