---
type: careers/interview
section: TypeScript
topic: Безопасность типов
status: ready
created: 2026-11-16
tags: [interview, typescript, any, assertion]
---

# Использование any и as

## Краткий ответ (30 секунд)

`any` и `as` — **аварийные выходы**, которые отключают проверку. `any` выключает её насовсем (и заражает окружение), `as` — **утверждение типа** в конкретном месте: «поверь мне, это `User`», без проверки в рантайме. `as` не преобразует данные (в отличие от `Number(x)`), не безопасен и не валидирует. Допустимо: миграция JS→TS, узкие места с плохими типами, `as const`, обход test-моков. Недопустимо: данные с сервера, неизвестные формы, «сделать чтобы компилировалось». Правильная альтернатива — сужение, предикаты (`value is T`) и схемы (`zod`/`valibot`).

## Подробно

### `as` — что это

```ts
const el = document.querySelector("#app") as HTMLDivElement;
el.innerHTML = ""; // работать будет, пока #app действительно div
```

- Это **type assertion**, а не приведение. В JS ничего не происходит; в рантайме `el` может быть `null` или `span`.
- Нельзя «утверждать» полностью несовместимые типы:

```ts
const n = "1" as number; // ❌ Ошибка: typescript видит несовместимость
const n2 = "1" as unknown as number; // ✅ обход — но это красный флаг
```

- С `as` отпадает проверка **свойств**: `(obj as User).email` не спасёт от `undefined`.

### Безопасные формы `as`

```ts
// 1. as const — литералы вместо широты
const config = { env: "prod" } as const;

// 2. сужение после проверки
if (el instanceof HTMLDivElement) {
  el.innerHTML = "";   // уже без as
}

// 3. DOM-хелпер с проверкой
function getElement<T extends Element>(selector: string): T {
  const el = document.querySelector<T>(selector);
  if (!el) throw new Error(`Not found: ${selector}`);
  return el;
}
```

### Плохие формы

```ts
// ❌ данные с сервера
const user = (await res.json()) as User;

// ❌ обход узкого типа
const id = data.id as string; // если id — number, ошибка только в проде

// ❌ «any в одежде as»
const x = value as any as string;
```

### Правильное: сужение

```ts
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    typeof (value as { id: unknown }).id === "number"
  );
}

const data: unknown = await res.json();
if (isUser(data)) {
  console.log(data.name); // ✅
}
```

- **Type predicate** (`value is User`) — проверка, которая сужает тип.
- Для сложных структур — схемы:

```ts
import { z } from "zod";
const UserSchema = z.object({ id: z.number(), name: z.string() });
const user = UserSchema.parse(await res.json()); // тип User + рантайм-проверка
```

### `satisfies` вместо `as`

```ts
// ❌ as меняет (широкий) тип и проверяет меньше
const routes = {
  home: "/",
  user: "/users/:id",
} as Record<string, string>; // потеряли литералы

// ✅ satisfies проверяет, но сохраняет точный тип
const routes = {
  home: "/",
  user: "/users/:id",
} satisfies Record<string, string>;
// typeof routes.home = "/"
```

`as const satisfies` — сочетание для конфигов.

### Где `any` оправдан

- Автогенерируемые типы из старого JS-кода.
- Очень динамичные библиотеки без типов (лучше найти `@types` или объявить `declare module`).
- Тестовые заглушки.
- Миграция: `// @ts-expect-error` с причиной — лучше, чем `any`.

### Контроль в проекте

- `strict: true` (`noImplicitAny`, `strictNullChecks`).
- ESLint: `@typescript-eslint/no-explicit-any`, `no-floating-promises`.
- `noUncheckedIndexedAccess` — `obj[key]` даёт `T | undefined`.
- Проверка `any` в ревью; `// @ts-expect-error` только с комментарием.

> [!tip] Как отвечать
> «`as` — это «поверь мне», без рантайм-эффекта. Для данных с сервера — `unknown` + zod или предикат. `satisfies` лучше `as` для конфигов. `any` допустим точечно, с комментарием и планом убрать».

## Связанные вопросы

- [[any, unknown и never]]
- [[Утилитарные типы]]
- [[interface и type]]

## Ресурсы

- TS handbook `satisfies`: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html
- zod: https://zod.dev/
