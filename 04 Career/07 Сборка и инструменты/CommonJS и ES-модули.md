---
type: careers/interview
section: Сборка и инструменты
topic: Модули
status: ready
created: 2026-11-16
tags: [interview, modules, esm, cjs]
---

# CommonJS и ES-модули

## Краткий ответ (30 секунд)

**CommonJS (CJS)** — старый модульный формат Node.js: `require()` синхронный, возвращает `module.exports` **во время выполнения**, модули кэшируются, циклы дают частичный экспорт. **ESM** — стандарт ECMAScript: `import`/`export`, **статический анализ на этапе загрузки**, live bindings (импорт — живая ссылка), асинхронная загрузка, tree-shaking. В браузере — только ESM (`<script type="module">`); в Node — оба, но ESM появился позже и требует `"type": "module"` или `.mjs`. Vite/ современные бандлы переходят на ESM; CJS остаётся для старых пакетов.

## Подробно

### Синтаксис

```js
// CJS
const fs = require("fs");
module.exports = { read };
exports.read = read;

// ESM
import fs from "node:fs";
export function read() {}
export default { read };
```

### Ключевые различия

| | CommonJS | ESM |
|---|---|---|
| Загрузка | синхронная, runtime | асинхронная, статическая |
| Статический анализ | ❌ (можно требовать динамически) | ✅ (пути литеральные по спецификации, `import()` — динамика) |
| Live bindings | нет (копия значения) | да (ссылка на экспорт) |
| `this` модуля | `module.exports` | `undefined` |
| `__dirname` | есть | нет (используйте `import.meta.dirname`) |
| Tree-shaking | плохо | хорошо |
| Топ-level await | ❌ | ✅ (ESM) |
| Порядок выполнения | по `require` | по импортам, «поднимается» |

### Live bindings — важный вопрос

```js
// counter.js (ESM)
export let count = 0;
export function inc() { count++; }

// main.js
import { count, inc } from "./counter.js";
inc();
console.log(count); // 1 — живая ссылка
```

```js
// CJS
let count = 0;
module.exports = { count, inc: () => count++ };
// в потребителе count останется 0 — скопировано
```

### Циклические зависимости

- CJS: вернёт частично заполненный `module.exports` — может быть `undefined`.
- ESM: TDZ/«cannot access before initialization» при обращении к неинициализированной привязке.

Оба случая — сигнал архитектурной проблемы; выносить общий код.

### `import()` — динамический импорт

```js
const { default: Chart } = await import("./chart.js"); // async, можно в CJS
```

Работает в обоих форматах; создаёт отдельный чанк в бандлере.

### ESM в браузере

```html
<script type="module" src="/main.js"></script>
```

- Строгий режим по умолчанию.
- `defer`-подобное поведение, `import`-пути с расширениями (без бандлера).
- `import maps` — сопоставление имён пакетов URL.

### Node: `.mjs` / `.cjs` / `"type"`

- `"type": "module"` в `package.json` — `.js` = ESM.
- `.mjs` — всегда ESM, `.cjs` — всегда CJS.
- ESM может импортировать CJS (`import x from "cjs-pkg"` — default = `module.exports`), **CJS не может через `require` импортировать ESM** до Node 22; в новых Node `require(esm)` поддерживается для синхронных ESM без top-level await — важное изменение.
- `exports` в `package.json` — условный экспорт (`import`/`require`/`types`).

```json
{
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs"
    }
  }
}
```

### Полностью ли исчезает CJS

Нет: серверный legacy, старые пакеты, инструменты. Но новые библиотеки — ESM, Vite 7+ вообще только ESM.

> [!warning] Подвох
> «Можно ли `require` ESM?» — исторически нельзя (`ERR_REQUIRE_ESM`); в новых Node — `require(esm)` работает для синхронных модулей без top-level await. На интервью лучше сказать: «Стандартно — нет, нужен `import()`; современный Node добавляет поддержку для синхронного ESM».

## Связанные вопросы

- [[Как работает сборка приложения]]
- [[Vite и Webpack]]
- [[package.json и версионирование]]

## Ресурсы

- Node ESM docs: https://nodejs.org/api/esm.html
- MDN modules: https://developer.mozilla.org/ru/docs/Web/JavaScript/Guide/Modules
