---
type: careers/interview
section: Vue
topic: Компоненты
status: ready
created: 2026-11-16
tags: [interview, vue, provide, inject]
---

# provide и inject

## Краткий ответ (30 секунд)

`provide(key, value)` в родителе и `inject(key, default?)` в любом потомке — способ отдать зависимость **через дерево** без прокидывания пропсов на каждом уровне (**props drilling**). Используется для: темы, i18n, конфигурации, DI-сервисов, контекста формы/таблицы. В Composition API — `provide`/`inject` из `vue` (только в `setup`/`beforeCreate`), реактивность — если передать `ref`/`reactive`/`computed`. Ключ — строка или `Symbol` (лучше). **Минусы**: неявная зависимость (труднее найти источник), нет обязательного контракта (в TS — `InjectionKey<T>`), риск «скрытых глобалов» и тестирования.

## Подробно

### Базово

```vue
<!-- App.vue -->
<script setup>
import { provide, ref, readonly } from "vue";
const theme = ref("light");
provide("theme", { theme, toggle: () => (theme.value = theme.value === "light" ? "dark" : "light") });
</script>
```

```vue
<!-- DeepChild.vue -->
<script setup>
import { inject } from "vue";
const { theme, toggle } = inject("theme", { theme: ref("light"), toggle: () => {} });
</script>
```

### Типизированный ключ (TS-паттерн)

```ts
// keys.ts
import type { InjectionKey, Ref } from "vue";

export interface ThemeApi {
  theme: Ref<"light" | "dark">;
  toggle: () => void;
}

export const ThemeKey: InjectionKey<ThemeApi> = Symbol("theme");
```

```ts
provide(ThemeKey, { theme, toggle });
const { theme } = inject(ThemeKey)!; // теперь без угадывания строк
```

### Реактивность

- Передали `ref` — потомки видят изменения (реагируют на `.value`).
- Передали `reactive` — то же.
- Передали **примитив** — получили снимок, не обновится.
- Перезаписали `provide` в дочернем провайдере — потомки получают **ближайший** (можно локально переопределять тему).
- `readonly` — скрыть возможность менять состояние из потомков.

### `inject` с задержкой / `injection` в композаблах

- `inject` работает только во время `setup` (синхронно). Для ленивых случаев — обёртка-композабл `useTheme()`, который сам вызывает `inject`.
- Дефолтное значение: `inject(key, defaultValue)` — избегает `undefined`.

### Когда уместно

| Кейс | Почему |
|---|---|
| Тема/i18n | почти не меняется, нужно везде |
| Сервис API/логгер | DI без глобальных переменных |
| Контекст формы (`FormProvider`) | поля внутри знают форму |
| Контекст таблицы/списка | строки видят конфигурацию |
| Плагин/библиотека | передача «настроек» без пропсов |

### Когда не надо

- Одно-два уровня вложенности — обычные пропсы.
- Часто меняющееся глобальное состояние — Pinia (инструменты, devtools, подписки).
- «Пробрасываю всё через provide, потому что не хочу разбираться» — неявные зависимости ломают трассировку.

### Тестирование

Provide можно замокать в тесте:

```js
mount(Child, { global: { provide: { [ThemeKey as any]: mockTheme } } });
```

### Nuxt/Vue-плагины

`app.provide(key, value)` в плагине — глобально для всех компонентов (без `provide`-цепочки). Это и есть «глобальная зависимость»; использовать осознанно.

> [!warning] Подвох
> «Почему `inject` вернул `undefined`?» — вы вызываете его вне `setup` (например, в обработчике клика или после `await` без `getCurrentInstance`) или провайдера нет выше. Решение: обёртка-композабл с `inject` в `setup`, дефолт, `Symbol`-ключ для ранней диагностики.

## Связанные вопросы

- [[19 Глобальные компоненты Vue]]
- [[20 Доступ к дочернему компоненту]]
- [[06 Слоты]]

## Ресурсы

- Vue provide/inject: https://vuejs.org/guide/components/provide-inject
