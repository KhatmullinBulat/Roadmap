---
type: careers/interview
section: Сборка и инструменты
topic: Бандлеры
status: ready
created: 2026-11-16
tags: [interview, vite, webpack, build]
---

# Vite и Webpack

## Краткий ответ (30 секунд)

**Vite** — dev-сервер на **нативных ESM**: файлы отдаются по мере запроса, трансформация esbuild/Oxc, HMR через WebSocket. В prod (Vite 8, март 2026) единственный бандлер — **Rolldown** (Rust, совместимый с Rollup plugin API), раньше было разделение esbuild (dev) + Rollup (build). Плюсы: мгновенный старт, простой конфиг, ESM-first. **Webpack** — бандлер с **полным графом на старте**, мощной системой лоадеров/плагинов, `Module Federation`, тонкой настройкой. В dev-режиме собирает всё (медленнее старт), но предсказуем и «всё есть». Сейчас Vite — дефолт для новых SPA; Webpack — в legacy/крупных корпоративных проектах.

## Подробно

### Vite: как работает dev

1. Браузер запрашивает `index.html` → Vite подставляет `<script type="module" src="/src/main.tsx">`.
2. Браузер запрашивает `main.tsx` → Vite **трансформирует на лету** (esbuild/Oxc) и подменяет импорты на URL.
3. Зависимости из `node_modules` **предбандлятся** (`optimizeDeps`) в один чанк — чтобы не было сотен запросов.
4. HMR через WebSocket: изменился модуль — обновляется только он и зависимые (`accept`).

Плюсы: старт за миллисекунды, независимо от размера проекта; нет «5 минут компиляции».

### Vite: prod

- **Vite 8** — Rolldown (Rust) + Oxc: в 10–30 раз быстрее Rollup на бенчмарках, plugin API Rollup-совместим.
- **До Vite 8** — esbuild (dev) + Rollup (prod); `rolldown-vite` был техническим превью для Vite 7.
- Настройки: `build.target` (по умолчанию Baseline widely available, Chrome 111+/Safari 16.4+ в Vite 8), `rollupOptions`, `manualChunks` → теперь `advancedChunks`.
- Node.js 20.19+/22.12+ (ESM-only пакет).

```ts
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  resolve: { alias: { "@": "/src" } },
  build: {
    target: "baseline-widely-available",
    sourcemap: true,
    rolldownOptions: {
      output: { advancedChunks: { groups: [{ name: "vendor", test: /node_modules/ }] } },
    },
  },
});
```

### Webpack: как работает

1. Собирает граф **сразу**: парсит entry, применяет `rules` (лоадеры: `babel-loader`, `ts-loader`, `css-loader`), резолвит.
2. Оптимизации: `splitChunks`, `runtimeChunk`, `SideEffectsFlag`.
3. Dev: `webpack-dev-server` + HMR; prod: `mode: "production"`.

```js
// webpack.config.js (упрощённо)
module.exports = {
  mode: "production",
  entry: "./src/index.tsx",
  module: {
    rules: [
      { test: /\.tsx?$/, use: "ts-loader", exclude: /node_modules/ },
      { test: /\.css$/, use: ["style-loader", "css-loader"] },
    ],
  },
  plugins: [new HtmlWebpackPlugin({ template: "./index.html" })],
  optimization: { splitChunks: { chunks: "all" } },
};
```

### Сравнение

| | Vite | Webpack |
|---|---|---|
| Dev-старт | мгновенно (ESM + on-demand) | собирает весь граф |
| Prod-бандлер | Rolldown (Rust) | свой (JS) |
| Конфиг | минимальный | большой, гибкий |
| Экосистема лоадеров | плагины Rollup | огромная |
| Module Federation | ограниченно/через плагины | нативно |
| Нужен для legacy-браузеров | нужны отдельные полифилы | из коробки |
| HMR | да | да |

### Когда что

- **Vite** — новый проект, SPA, библиотеки, Astro/Vitest/Storybook — всё Vite-нативно.
- **Webpack** — legacy, сложные корпоративные сборки, Module Federation, нетипичные требования (несколько entry, микросервисные бандлы).
- **Rspack/Turbopack/esbuild** — альтернативы «быстрее Webpack/Vite» с частичной совместимостью; на интервью достаточно упомянуть.

### Подвох

> «Почему Vite в прод не использует нативные ESM?» — сотни запросов без предбандлинга и минификации плохо работают на реальных сетях; prod нужен единый оптимизированный бандл (Rolldown). Dev — нативный ESM, prod — бандл.

## Связанные вопросы

- [[Как работает сборка приложения]]
- [[CommonJS и ES-модули]]
- [[package.json и версионирование]]

## Ресурсы

- Vite: https://vite.dev/
- Vite 8: https://vite.dev/blog/announcing-vite8
- Webpack: https://webpack.js.org/
