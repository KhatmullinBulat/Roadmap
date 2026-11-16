---
type: careers/interview
section: Тестирование
topic: E2E
status: ready
created: 2026-11-16
tags: [interview, e2e, playwright, cypress]
---

# E2E тесты и Playwright

## Краткий ответ (30 секунд)

**E2E** проверяет сценарий целиком в **настоящем браузере** (реальный бэкенд или его контейнер): «пользователь пришёл → вошёл → оформил заказ». **Playwright** — современный инструмент: работает в Chromium/Firefox/WebKit, авто-ожидания (locator ждёт элемент), изоляция контекста, трейсы/видео, параллельность, `storageState` для авторизации, visual-снапшоты. **Cypress** — DX попроще, работает в одном браузере-«раннере» (Chromium/Firefox/WebKit), но сценарии ограничениями (несколько вкладок/доменов — с оговорками). Правило: e2e — немного, только критичные пути; всё остальное — интеграционные тесты.

## Подробно

### Playwright: тест

```ts
import { test, expect } from "@playwright/test";

test("оформление заказа", async ({ page }) => {
  await page.goto("/");
  await page.getByRole("link", { name: "Войти" }).click();
  await page.getByLabel("Email").fill("anna@example.com");
  await page.getByLabel("Пароль").fill("secret");
  await page.getByRole("button", { name: "Войти" }).click();

  await expect(page.getByRole("heading", { name: "Личный кабинет" })).toBeVisible();

  await page.getByRole("button", { name: "Добавить в корзину" }).click();
  await expect(page.getByTestId("cart-count")).toHaveText("1");
});
```

- **`locator`** — ленивый «селектор»; **авто-ожидание** в действиях и `expect`.
- `getByRole`, `getByLabel`, `getByText` — доступные локаторы (тот же приоритет, что в RTL).
- `expect(...).toBeVisible()/toHaveText()/toHaveURL()` — с внутренним retry.
- **Не использовать `page.waitForTimeout`** — причина flaky-тестов.

### Изоляция и скорость

- **`test.use({ storageState })`** — сохранить авторизацию один раз (`playwright codegen` или `storageState` после логина), не логиниться в каждом тесте.
- **`projects`** — Chromium/Firefox/WebKit, мобильные viewport.
- **Параллельность** — `fullyParallel`, шардинг в CI (`--shard=1/3`).
- **Трейсы:** `trace: "on-first-retry"` → `npx playwright show-trace`.
- **Сеть:** `page.route()` — мокировать внешние сервисы (платежи).
- **`baseURL`** + `webServer` в конфиге — поднимает dev-сервер.

### Cypress: чем отличается

```js
it("добавляет в корзину", () => {
  cy.visit("/");
  cy.contains("button", "Купить").click();
  cy.get("[data-testid=cart-count]").should("have.text", "1");
});
```

- Работает «внутри» браузера: команды в цепочке, автоretry у `should`.
- Проще стартовать/отлаживать (интерактивный runner).
- Ограничения: кросс-доменные сценарии, несколько вкладок, фоновая работа — исторически проблемные места.
- Сейчас есть WebKit-экспериментальный и параллельность платная.

### Сравнение

| | Playwright | Cypress |
|---|---|---|
| Браузеры | Chromium, Firefox, WebKit | Chromium, Firefox, WebKit |
| Архитектура | CDP/протокол, вне браузера | внутри браузера |
| Параллельность | бесплатно | платно (Cypress Cloud) |
| Мультивкладки/домены | да | ограничено |
| Трейсы/видео | отлично | есть (видео) |
| DX | чуть сложнее | очень доступен |

### Почему e2e «flaky» и как бороться

1. **Ожидание времени** → заменять на ожидание состояния (locator, `expect`, ответ сети).
2. **Слишком много e2e** → покрывать бизнес-логику интеграционными.
3. **Общий state** → изолированные пользователи/данные через API-фикстуры.
4. **Гонки с бэкендом** → `waitForResponse`/`expect.poll`.
5. **Нет трейсов** → включить `trace` и видео.

### Что покрывать e2e

- Логин/регистрация.
- Оформление заказа/оплата (с моками платёжного шлюза).
- Критичные права доступа.
- Smoke: приложение открывается и главный экран работает.

### CI

```yaml
- run: npx playwright install --with-deps
- run: npx playwright test --shard=${{ matrix.shard }}/3
```

- Артефакты: `trace`, `screenshot` при падении.
- Запуск на PR — smoke-набор; полный — в nightly.

> [!tip] Как отвечать
> «E2E — про путь пользователя, а не про все фичи. Playwright: локаторы по роли, авто-ожидания, storageState, трейсы. Flaky лечим ожиданием состояния и изоляцией данных, а не `sleep`».

## Связанные вопросы

- [[Пирамида тестов и виды тестов]]
- [[React Testing Library]]
- [[CI CD и деплой]]

## Ресурсы

- Playwright: https://playwright.dev/
- Cypress: https://www.cypress.io/
