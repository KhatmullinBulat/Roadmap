---
type: careers/interview
section: Тестирование
topic: Инструменты
status: ready
created: 2026-11-16
tags: [interview, jest, vitest]
---

# Jest и Vitest

## Краткий ответ (30 секунд)

**Jest** — самый распространённый раннер: `describe`/`it`/`expect`, моки (`jest.fn`, `jest.mock`), снапшоты, jsdom-окружение, `jest --watch`, покрытие (istanbul/v8). Работает через свой трансформер (babel/ts-jest/swc). **Vitest** — «Vite-native» альтернатива: тот же API (`expect`, `describe`), но использует конфиг Vite, ESM «из коробки», быстрее стартует, поддерживает `vi.fn`/`vi.mock`, **Browser Mode** (стабильный с v4 — тесты в настоящем браузере) и визуальные снапшоты. Куда идти: Vite-проект — Vitest; старый Jest-проект — Jest 30 (ESM-поддержка, `jest.config.mts`).

## Подробно

### Общая структура

```js
import { describe, it, expect, vi, beforeEach } from "vitest"; // или jest

describe("calculateTotal", () => {
  beforeEach(() => vi.clearAllMocks());

  it("складывает цены", () => {
    expect(calculateTotal([{ price: 100 }, { price: 50 }])).toBe(150);
  });

  it("бросает на пустом массиве", () => {
    expect(() => calculateTotal([])).toThrow("empty");
  });
});
```

### Матчеры (важное)

```js
expect(a).toBe(b);            // Object.is — примитивы/ссылки
expect(obj).toEqual({...});   // глубокая проверка; undefined-свойства игнорируются
expect(obj).toStrictEqual();  // строже: прототипы и undefined-свойства учитываются
expect(arr).toContain(x);
expect(fn).toHaveBeenCalledWith(x);
expect(fn).toHaveBeenCalledTimes(1);
expect(p).resolves.toBe(x);
expect(p).rejects.toThrow("err");
expect(el).toBeInTheDocument(); // DOM-матчеры (jest-dom)
```

`toBe` для объектов — сравнение ссылок, поэтому чаще `toEqual`.

### Моки

```js
vi.fn();                          // шпион
vi.fn().mockReturnValue(42);
vi.spyOn(api, "get").mockResolvedValue(data);
vi.mock("./module", () => ({ x: 1 })); // хойстится
```

- `jest.mock`/`vi.mock` поднимается в начало файла (hoisting) — фабрика не видит переменные, объявленные после; решение — `vi.hoisted`.
- `mockResolvedValue`/`mockRejectedValue` для промисов.
- `clearAllMocks` (сброс вызовов), `resetAllMocks` (и реализаций), `restoreAllMocks` (spyOn).

### Снапшоты

```js
expect(renderTree).toMatchSnapshot();
expect(data).toMatchInlineSnapshot();
```

- Ловят неожиданные изменения.
- Не заменяют поведенческие тесты.
- Обновлять **осознанно** (`--update`), с ревью diff.
- Для компонентов лучше `toMatchInlineSnapshot()` — видно в файле.

### Асинхронность

```js
await waitFor(() => expect(el).toBeVisible());
await user.click(button);
await expect(api.get()).resolves.toEqual(data);
```

- Не забывать `await`, иначе тест проходит/падает нестабильно.
- Таймеры: `vi.useFakeTimers()` + `vi.advanceTimersByTime(1000)`; `jest.advanceTimersToFrame()` — для rAF.

### Среды

- **jsdom** / **happy-dom** — эмуляция DOM в Node, быстрее, но не настоящий браузер.
- **Browser Mode (Vitest 4)** — Playwright/WebDriverIO в реальном браузере: проверяет настоящие стили/layout/API.
- **node** — для утилит и сервера.

### Jest 30 и Vitest 4 (что нового)

- **Jest 30** (июнь 2025): ESM-поддержка, `jest.config.mts`, `using` с `jest.spyOn`, ускорение.
- **Vitest 4** (октябрь 2025): стабильный Browser Mode (`@vitest/browser-playwright`), визуальные регрессии (`toMatchScreenshot`), Playwright Traces, `toBeInViewport`.

### Когда что выбрать

- **Vite/React/Next/Vitest-совместимый стек** — Vitest: общий конфиг, одинаковые алиасы, быстрее.
- **Существующий Jest-проект** — не мигрировать без причины; Jest живой, ESM улучшен.
- **Нужны реальные браузеры и визуальные тесты** — Vitest Browser Mode или Playwright.

> [!warning] Подвох
> «`jsdom` = браузер?» — нет: он эмулирует DOM в Node, но не layout, CSS, настоящие API. Если тест зависит от размеров/стилей — нужен Browser Mode. `toBeVisible()` в jsdom не проверяет реальную видимость так, как браузер.

## Связанные вопросы

- [[React Testing Library]]
- [[Пирамида тестов и виды тестов]]
- [[E2E тесты и Playwright]]

## Ресурсы

- Vitest: https://vitest.dev/
- Jest: https://jestjs.io/
