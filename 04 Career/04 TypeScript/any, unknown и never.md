---
type: careers/interview
section: TypeScript
topic: Безопасность типов
status: ready
created: 2026-11-16
tags: [interview, typescript, any, unknown, never]
---

# any, unknown и never

## Краткий ответ (30 секунд)

**`any`** отключает проверку: с ним можно всё, и ошибки уходят в рантайм. **`unknown`** — «что-то, но пока неизвестно»: можно хранить, но нельзя использовать без **сужения** (`typeof`, `in`, `instanceof`, предикат). Это безопасная замена `any` на границах (входные данные, `catch`, `JSON.parse`). **`never`** — тип значений, которых **не может быть**: функция всегда бросает/не завершается, исчерпывающая проверка в `switch`, `Exclude`-результаты. `any` протекает по всему коду, `unknown` заставляет проверить, `never` — доказывает, что случай не оставлен.

## Подробно

### `any`

```ts
let x: any = "hello";
x.toUpperCase();
x.foo.bar;     // ошибок нет
x();           // тоже
```

- Используется для миграции JS→TS, для быстрых заглушек, для динамических API без типов.
- **Протекает**: `any` в функции → результат тоже `any`.
- `noImplicitAny` запрещает *неявный* any (параметры без типа), но `: any` явно — можно.
- `strict` включает `noImplicitAny`, но не запрещает явный.

### `unknown`

```ts
function parse(input: unknown) {
  if (typeof input === "string") {
    input.toUpperCase(); // ✅ после сужения
  }
  // input.foo — ❌ до сужения
}
```

- **Сужать обязательно**: `typeof`, `instanceof`, `in`, `Array.isArray`, пользовательский предикат.
- `unknown` — правильный тип для:
  - `catch (e: unknown)` (в TS 4.4+ catch — `unknown` при `useUnknownInCatchVariables`);
  - `JSON.parse` (де-факто `any`, но оборачивают в `unknown`);
  - `fetch().json()`;
  - параметров «любой вход».

```ts
try {
  risky();
} catch (e) {
  if (e instanceof Error) console.log(e.message);
  else console.log(String(e));
}
```

### `never`

```ts
function fail(msg: string): never {
  throw new Error(msg);
}

function assertNever(x: never): never {
  throw new Error(`Unexpected: ${x}`);
}

type Shape = { kind: "circle"; r: number } | { kind: "square"; s: number };

function area(shape: Shape) {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.r ** 2;
    case "square": return shape.s ** 2;
    default: return assertNever(shape); // если добавить "triangle", будет compile error
  }
}
```

- `never` — **пустое множество значений**. `never` совместим со всем (assignment из `never` разрешён), но в `never` нельзя присвоить ничего (кроме `never`).
- **Discriminated union** + `assertNever` = исчерпывающий `switch`.
- `never` как «функция не возвращает»: `throw`, бесконечный цикл.
- `Exclude<T, U>` и `ReturnType` могут давать `never`.
- В React `ReactNode` не путайте: `never` — пустой тип, не `null`.

### Таблица

| | `any` | `unknown` | `never` |
|---|---|---|---|
| Можно использовать до сужения | да | нет | — |
| Присвоить в `string` | да | нет | да |
| Присвоить из `string` | да | да | нет |
| Свойства | любые | только после сужения | — |
| Когда | legacy/миграция | входные данные | исчерпывающий switch, throw |
| Опасность | подавляет ошибки | почти безопасно | путают с `void` |

### Сравнение `unknown` и `any` на практике

```ts
const data: unknown = JSON.parse(responseText);

// ❌ «проверка» через as
const user = data as User;

// ✅ через схему
const user = UserSchema.parse(data);
```

### Частый подвох

`unknown` **не** эквивалентен `object | string | number | ...`. Это «совсем неизвестно». И `never` **не** равен `void`: `void` — функция вернула `undefined`, `never` — не вернула никогда.

> [!tip] Как отвечать
> «`any` выключает проверки и заражает. `unknown` — безопасно: пока не сузил, ничего не сделать. `never` — пустое множество: функция никогда не возвращается или исчерпывающий switch. На границе (JSON, catch) — `unknown` + валидация, а не `as`».

## Связанные вопросы

- [[Использование any и as]]
- [[extends в TypeScript]]
- [[Утилитарные типы]]

## Ресурсы

- TS handbook «Narrowing»: https://www.typescriptlang.org/docs/handbook/2/narrowing.html
- `never` и exhaustiveness: https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking
