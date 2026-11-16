---
type: careers/interview
section: Тестирование
topic: Компоненты
status: ready
created: 2026-11-16
tags: [interview, react, testing, rtl]
---

# React Testing Library

## Краткий ответ (30 секунд)

**RTL** — утилиты для тестирования компонентов «как пользователь»: рендер, поиск по **роли/тексту/label** (`getByRole`, `getByLabelText`), взаимодействие (`userEvent`), ожидание (`waitFor`, `findBy*`). Философия: «чем больше тест похож на использование, тем больше уверенности». Не тестировать внутренности (state, классы, приватные хуки) и не искать по `data-testid`, если можно по роли. Рядом — `@testing-library/jest-dom` (матчеры `toBeInTheDocument`, `toHaveAccessibleName`), `@testing-library/user-event` (реальные события), **MSW** (моки сети).

## Подробно

### Базовый тест

```jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

it("показывает ошибку при пустом email", async () => {
  const user = userEvent.setup();
  render(<Login onSubmit={vi.fn()} />);

  await user.click(screen.getByRole("button", { name: /войти/i }));

  expect(await screen.findByRole("alert")).toHaveTextContent(/введите email/i);
});
```

- `getBy*` — сразу, падает если нет; `queryBy*` — `null` если нет; `findBy*` — асинхронно (ждёт).
- `getAllBy*` — массив; `*ByRole` — предпочтительный запрос.
- `screen` — «вся страница», не привязывает к `container`.

### Запросы по приоритету

1. `getByRole` (+ `name`) — доступность: «кнопка Войти».
2. `getByLabelText` — поля формы.
3. `getByPlaceholderText` — только если placeholder заменяет label (плохо).
4. `getByText` — текст, не input.
5. `getByDisplayValue`, `getByAltText`, `getByTitle`.
6. `getByTestId` — последний вариант, когда ничего не подходит.

### `userEvent` vs `fireEvent`

```js
// ✅ реальное поведение: фокус, ввод по символам, события клавиатуры
await user.type(input, "anna@example.com");
await user.click(button);
await user.keyboard("{Escape}");

// ⚠️ fireEvent — одно синтетическое событие, без фокуса и detail
fireEvent.change(input, { target: { value: "x" } });
```

`userEvent.setup()` — обязателен (порядок и таймеры).

### Асинхронность

```js
// ✅ дождаться появления
expect(await screen.findByText("Готово")).toBeInTheDocument();

// ✅ дождаться условия
await waitFor(() => expect(screen.getByRole("status")).toHaveTextContent("1"));

// ❌ sleep
await new Promise((r) => setTimeout(r, 500));
```

- `findBy*` — если нужен элемент, который появится.
- `waitFor` — если проверка не сводится к появлению (например, изменение текста).
- Ошибки, выброшенные в `waitFor`, ретраятся до таймаута.

### Моки сети — MSW

```js
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";

const server = setupServer(
  http.get("/api/user", () => HttpResponse.json({ name: "Аня" })),
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

- Перехват на уровне сети (`fetch`/XHR) — не надо мокать модули.
- Работает и в браузере (Service Worker), и в Node.
- Позволяет тестировать ошибки: `HttpResponse.json({}, { status: 500 })`.

### Что тестировать в компоненте

- Отображение пропсов/данных.
- Взаимодействия → изменения (клик → колбэк/состояние).
- Ошибки/загрузка/пустые состояния.
- Доступность: `toHaveAccessibleName`, роль.

### Антипаттерны

- `container.querySelector(".btn")` — тест на классы.
- Тест на внутренний `useState`/структуру DOM.
- `getByTestId("btn-1")` везде.
- Мок `useState`/хуков вместо тестирования результата.
- Снапшот всего компонента вместо проверки поведения.
- `act`-ворнинги: игнорирование → нестабильные тесты. Обновление вне `act` обычно из-за асинхронного эффекта без `await`.

### Не забудь `cleanup`

RTL автоматически вызывает `cleanup` после каждого теста при использовании setup-файла (Vitest/Jest). Если тесты «протекают» между файлами — проверьте setup.

> [!tip] Как отвечать
> «Тестирую по ролям и labelText, как пользователь. `userEvent` вместо `fireEvent`, `findBy`/`waitFor` вместо sleep. Сеть — MSW, чтобы не мокать модули. Внутренности компонента не проверяю».

## Связанные вопросы

- [[Jest и Vitest]]
- [[Пирамида тестов и виды тестов]]
- [[Доступность]]

## Ресурсы

- Testing Library: https://testing-library.com/docs/react-testing-library/intro
- MSW: https://mswjs.io/
