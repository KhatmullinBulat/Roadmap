---
type: careers/interview
section: TypeScript
topic: Компиляция
status: ready
created: 2026-11-16
tags: [interview, typescript, compilation]
---

# Во что компилируются type и interface

## Краткий ответ (30 секунд)

**Ни во что.** Типы, интерфейсы, алиасы и аннотации существуют только на этапе проверки и **стираются** при компиляции (type erasure). В JS-выходе нет ни `type`, ни `interface`, ни проверок — остаются только значения. С `declaration: true` рядом появляется `.d.ts` — текст, а не рантайм. Исключения — конструкции, которые создают значения: `enum`, `namespace` (с значениями), параметры с инициализаторами (JS по умолчанию), декораторы (раньше — рантайм-обёртки, сейчас — через компилятор), `satisfies`/`as` — тоже стираются.

## Подробно

### Что исчезает

```ts
interface User {
  id: number;
  name: string;
}

type UserId = string;

function greet(user: User): string {
  return `Hi, ${user.name}`;
}

const id: UserId = "u1";
```

→

```js
function greet(user) {
  return `Hi, ${user.name}`;
}
const id = "u1";
```

- `interface`, `type`, аннотации, параметры типов (`<T>`), `implements`, `as`, `satisfies`, `declare` — **исчезают**.
- `!` (non-null assertion), `?.` — `?.` **остаются** как JS-операторы, `!` исчезает.
- `import type` / `export type` — удаляются целиком (важно для `isolatedModules`/`verbatimModuleSyntax`).

### Что остаётся

| Конструкция | Что в JS |
|---|---|
| `enum` | IIFE + объект |
| `const enum` | инлайн-значения |
| `namespace` с значениями | IIFE |
| `class` + модификаторы | класс (с приватностью через `#`, если используется) |
| значения по умолчанию параметров | JS default params |
| декораторы (новый стандарт) | рантайм-вызовы через помощник |
| типы в JSDoc | комментарии (если `checkJs`/`allowJs`) |

### Почему это важно

1. **Нет рантайм-валидации.** `JSON.parse` вернёт `any`; `as User` — лишь обещание. Данные с сервера надо валидировать (zod/valibot/io-ts).
2. **Нет рефлексии.** Нельзя проверить «а это тип `User`?» — тип потерян; нужны предикаты (`value is User`) или схемы.
3. **Tree-shaking.** Типы не тянут код — это их преимущество по сравнению с enum.
4. **`isolatedModules`.** Каждый файл компилируется отдельно (Vite/esbuild/SWC). Конструкции, которым нужен полный граф (`const enum`, реэкспорт типов без `export type`), ломаются.
5. **`.d.ts`.** При `declaration: true` генерируются декларации — они нужны для публикации библиотек и для IDE, но в JS не идут.

### Проверка на практике

```js
// user.ts
export interface User { name: string }
export const createUser = (name: string): User => ({ name });
```

```bash
npx tsc --target es2022 user.ts
```

В `user.js` останется только `createUser`, без `User`.

### Мини-ответ

> «Type и interface — compile-time only, их стирает type erasure. В рантайме нет ни типов, ни проверок. Поэтому валидация — на границе (zod), а не `as`. Декларации в `.d.ts` — это текст для инструментов, не код».

## Связанные вопросы

- [[interface и type]]
- [[Enum и компиляция]]
- [[any, unknown и never]]
- [[Использование any и as]]

## Ресурсы

- TS handbook Compiler Options (`isolatedModules`, `declaration`): https://www.typescriptlang.org/tsconfig
