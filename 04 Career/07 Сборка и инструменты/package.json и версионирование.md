---
type: careers/interview
section: Сборка и инструменты
topic: Экосистема
status: ready
created: 2026-11-16
tags: [interview, npm, semver]
---

# package.json, SemVer и lock-файлы

## Краткий ответ (30 секунд)

`package.json` — манифест: `name`, `version`, `scripts`, `dependencies` (в рантайме), `devDependencies` (сборка/тесты), `peerDependencies` (ожидание от хост-проекта, например React), `exports`/`main`/`module`/`types` (точки входа), `engines`. **SemVer**: `MAJOR.MINOR.PATCH` — мажор ломает API, минор добавляет функционал обратно совместимо, патч — багфиксы. **Lock-файл** (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`) фиксирует точные версии всего дерева; для CI — `npm ci`/`pnpm install --frozen-lockfile`, не `npm install`. `~1.2.3` — патч, `^1.2.3` — минор (ближайший), `1.2.3` — точно.

## Подробно

### Типы зависимостей

| Поле | Когда |
|---|---|
| `dependencies` | нужно в рантайме приложения |
| `devDependencies` | сборка, тесты, линтеры |
| `peerDependencies` | библиотека требует React/Vue от хоста |
| `optionalDependencies` | может отсутствовать (esbuild/rollup платформенные биндинги) |
| `bundledDependencies` | включено в пакет |

Для библиотеки с React: `peerDependencies: { react: ">=18" }` и не дублировать в `dependencies` (иначе две копии React).

### Семантическое версионирование

```
1.4.2
│ │ └─ PATCH: багфиксы
│ └─── MINOR: новая функциональность, обратно совместимо
└───── MAJOR: несовместимые изменения
```

- `^1.4.2` — `>=1.4.2 <2.0.0`.
- `~1.4.2` — `>=1.4.2 <1.5.0`.
- `1.4.x`, `*`, `latest` — избегать.
- `0.x` — формально «до стабильного релиза»; `^0.4.0` в npm = `<0.5.0` (мажор = минор для 0.x).
- Prerelease: `2.0.0-beta.1` не подпадёт под `^1`.

### Lock-файл

- Фиксирует **всё дерево** (резолв, integrity-хеши).
- Нужен в репозитории; удалять — значит «у всех будет разное».
- `npm ci` — по lock, без изменения; быстрее и воспроизводимо.
- `npm install` — может обновить lock и версии по диапазонам.
- В библиотеках lock тоже полезен, но публикуется только `package.json`.

### `exports` и точки входа

```json
{
  "name": "ui-kit",
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./styles.css": "./dist/styles.css"
  },
  "peerDependencies": { "react": ">=18" },
  "sideEffects": ["**/*.css"]
}
```

- **`exports` блокирует** доступ к «внутренним» путям: `import "ui-kit/dist/secret"` не пройдёт, если не объявлен.
- `"type": "module"` — `.js` в пакете = ESM.
- `sideEffects` — для tree-shaking.

### `scripts` и `npx`

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "test": "vitest run",
    "lint": "eslint ."
  }
}
```

- `npm run` работает из `node_modules/.bin`.
- `npx vite` — запуск пакета без установки глобально.

### Мониторинг зависимостей

- `npm audit` / `pnpm audit` — уязвимости.
- `npm outdated`, `npm update`.
- Dependabot/Renovate — автоматические PR.
- `npm ls react` — дубли в дереве.
- «Мёртвые» зависимости — `depcheck` (осторожно с false-positive).

### Конфликты версий

- Две версии React — `Invalid hook call`.
- `peerDependencies` + `overrides` (npm) / `resolutions` (yarn) для принудительной версии:

```json
"overrides": { "lodash": "4.17.21" }
```

> [!warning] Подвох
> «`npm install` в CI — нормально?» — нет: без `npm ci` lock-файл может измениться, и сборка будет невоспроизводимой. Другое классическое: `^` в библиотеке — пользователь может получить «новый минор», который ломает проект из-за плохой совместимости.

## Связанные вопросы

- [[CommonJS и ES-модули]]
- [[Vite и Webpack]]
- [[Монорепозиторий и пакетный менеджер]]

## Ресурсы

- npm docs: https://docs.npmjs.com/cli/v10/configuring-npm/package-json
- SemVer: https://semver.org/
