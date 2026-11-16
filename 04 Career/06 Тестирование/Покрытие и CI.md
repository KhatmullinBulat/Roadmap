---
type: careers/interview
section: Тестирование
topic: Инфраструктура
status: ready
created: 2026-11-16
tags: [interview, testing, ci, coverage]
---

# Покрытие, CI и проверки качества

## Краткий ответ (30 секунд)

**Покрытие** (coverage) измеряет, какие строки/ветки/функции исполнились: `lines`, `statements`, `branches`, `functions`. 100% не значит «хорошо» — можно пройти строку без проверки ошибок. В CI: **lint → typecheck → unit → build → e2e (smoke)**, отчёт покрытия в PR, запрет падать при падении тестов, фиксированные версии Node, кэш зависимостей. Качество — это не только тесты: типы, линтер, ревью, контрактные тесты API, мониторинг в проде.

## Подробно

### Coverage

```bash
npx vitest run --coverage
```

- **Statements** — выполненные инструкции.
- **Branches** — обе ветки `if`/тернарника/`switch` (важнее строк).
- **Functions** — вызванные функции.
- **Lines** — строки.

Пороги:

```ts
// vitest.config.ts
test: {
  coverage: {
    provider: "v8",
    reporter: ["text", "lcov", "html"],
    thresholds: { lines: 80, branches: 70, functions: 80 },
  },
}
```

- Порог выставляют на **критичные модули**, не глобально «100».
- Игнорировать: миграции, типы, конфиги.
- Провайдер **v8** — быстрее, **istanbul** — точнее по веткам.

### CI-пайплайн

```yaml
jobs:
  check:
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm run test -- --coverage
      - run: npm run build
  e2e:
    needs: check
    steps:
      - run: npx playwright install --with-deps
      - run: npx playwright test --grep @smoke
```

- `npm ci` (не `install`) — воспроизводимый lock-файл.
- Fail fast: линт/типы вперёд, тесты и сборка после.
- Кэш зависимостей — иначе каждый прогон медленный.
- Артефакты: coverage, Playwright trace, скриншоты.

### Что ещё входит в «качество»

- **Контрактные тесты** API (OpenAPI/zod-схемы на границе).
- **E2E smoke** на staging перед деплоем.
- **Bundle-size лимит** (`size-limit`, `bundlemon`) — не раздулся ли бандл.
- **Lighthouse CI** — метрики производительности/a11y как проверка.
- **Линтер доступности** (`eslint-plugin-jsx-a11y`).
- **Dependabot/Renovate** — обновления зависимостей.
- **Мониторинг в проде** (Sentry, RUM) — тесты не ловят всё.

### Принципы

1. **Тесты должны быть быстрыми**, иначе их не запускают.
2. **Стабильность**: flaky-тест хуже отсутствующего — ему не верят.
3. **Локальный запуск** = CI: одинаковые команды.
4. **Не блокировать merge** из-за покрытия на всём репозитории — только на изменённых файлах (diff coverage).
5. **Фиксированные версии** Node и пакетов.

### Покрытие в PR

- Комментарий с diff-покрытием (Codecov, Coveralls).
- «Новый код не покрыт» — повод для ревью, но не всегда блокер.
- Legacy-модули исключать явно, а не тянуть в общий порог.

> [!tip] Как отвечать
> «Покрытие — сигнал, а не цель. В CI — lint → types → tests → build → e2e smoke, с артефактами и кэшем. Пороги — на критичные модули; плюс bundle-size и Lighthouse, потому что тесты не ловят деградацию производительности».

## Связанные вопросы

- [[Пирамида тестов и виды тестов]]
- [[E2E тесты и Playwright]]
- [[Vite и Webpack]]

## Ресурсы

- Vitest Coverage: https://vitest.dev/guide/coverage
- GitHub Actions: https://docs.github.com/actions
