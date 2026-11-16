---
type: careers/interview
section: Сборка и инструменты
topic: Деплой
status: ready
created: 2026-11-16
tags: [interview, build, ci, deploy]
---

# CI/CD и проверки качества

## Краткий ответ (30 секунд)

**CI** — на каждый PR: установка (с lock-файлом) → lint → typecheck → unit-тесты → build → e2e (smoke) → проверка бандла/метрик. **CD** — деплой артефакта: prerelease (preview/staging) на PR, prod после merge, с откатом. Ключевые принципы: **один артефакт → все окружения**, кэш зависимостей и задач, fail fast, артефакты при падении (trace/скриншоты), фиксированные версии. Не деплоить из локальной машины; не ломать `main`.

## Подробно

### Типичный пайплайн

```yaml
name: ci
on: [pull_request]

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm run test -- --coverage
      - run: npm run build
      - run: npx size-limit

  e2e:
    needs: verify
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test --grep @smoke
      - uses: actions/upload-artifact@v4
        if: failure()
        with: { name: playwright-traces, path: test-results }
```

### Сборка и артефакты

- Сборка **один раз**, деплоится тот же артефакт.
- Хешированные имена файлов + `immutable` для ассетов.
- `index.html` — короткий кэш.
- Деплой: атомарно (новая папка + переключение), откат к предыдущей версии.

### Среды

| Среда | Когда | Данные |
|---|---|---|
| Preview (per-PR) | каждый PR | staging/сеансовые |
| Staging | merge/preprod | приближенные к прод |
| Prod | релиз | реальные |

- Feature flags — выкатывать код без включения.
- Канареечный/поэтапный деплой — процент трафика.
- Миграции БД — отдельно от кода, назад совместимо.

### Проверки «за пределами тестов»

- **Bundle size** — `size-limit`.
- **Lighthouse CI** — метрики/a11y на ключевых страницах.
- **Security** — `npm audit`, Dependabot, secret scanning.
- **Accessibility** — axe в e2e.
- **Changelog** — Changesets/PR-теги.
- **Dependency updates** — Renovate/Dependabot отдельными PR.

### Кэш и скорость CI

- Кэш `node_modules`/pnpm store.
- Turborepo/Nx remote cache для монорепо.
- Шардинг тестов (единицы — в параллель).
- Не запускать всё на каждый коммит: `affected`-запуски.

### Секреты

- Никогда в репозитории/логах.
- Ключи — в секретах CI, прокидывать только на нужный шаг.
- `NODE_ENV=production`, `VITE_*` — помнить, что всё в клиентском бандле публично.

### Что говорить на вопрос «как выкатываете»

> «PR → preview-деплой и проверки (lint/types/tests/build/size). После merge — staging smoke, затем prod с ручным подтверждением или канарейкой. Артефакт один и тот же, откат — переключением на предыдущую сборку. Миграции — обратимыми шагами».

> [!warning] Подвох
> «VITE_/NEXT_PUBLIC_ переменные — секреты?» — нет: всё, что попадает в клиентский бандл, публично. Секреты живут на сервере/в прокси-функциях.

## Связанные вопросы

- [[Покрытие и CI]]
- [[Vite и Webpack]]
- [[Монорепозиторий и пакетный менеджер]]

## Ресурсы

- GitHub Actions: https://docs.github.com/actions
- Lighthouse CI: https://github.com/GoogleChrome/lighthouse-ci
