---
type: careers/interview
section: Vue
topic: State
status: ready
created: 2026-11-16
tags: [interview, vue, pinia]
---

# Store и Pinia: state, getters, actions

## Краткий ответ (30 секунд)

**Store** — общий источник истины для данных, которые нужны **нескольким** компонентам (сессия, корзина, справочники, фильтры между роутами). Если данные нужны одному компоненту и его детям — хватит локального `ref` и пропсов. **Pinia**: **`state`** — данные, **`getters`** — вычисляемые значения (кэш, как `computed`), **`actions`** — методы, в том числе асинхронные. Всё реактивно; компонент получает стор через `useStore()`, и Vue сам отслеживает использованные поля. Дух Pinia — без «мутаций»: можно менять state напрямую, но с привычками (действия) удобнее.

## Подробно

### Когда нужен стор

- Данные из разных **несвязанных** частей дерева (header, роут, модалка).
- Состояние, которое переживает навигацию (корзина, черновик).
- Кэш справочников и API-ответов.
- Глобальные настройки/тема (хотя для темы хватит `provide`/CSS-переменных).

### Когда не нужен

- Локальное состояние формы/модалки.
- Данные, нужные только родителю и детям — `props`/`emit`.
- Просто «shared util» — композабл с `ref` вне стора (пока не нужны devtools).

### Pinia-стор

```ts
// stores/cart.ts
import { defineStore } from "pinia";
import { computed, ref } from "vue";

export const useCartStore = defineStore("cart", () => {
  const items = ref<{ id: number; title: string; price: number }[]>([]);

  const total = computed(() => items.value.reduce((s, i) => s + i.price, 0));
  const count = computed(() => items.value.length);

  async function add(id: number) {
    const product = await api.getProduct(id);
    items.value.push(product);
  }

  function remove(id: number) {
    items.value = items.value.filter((i) => i.id !== id);
  }

  return { items, total, count, add, remove };
});
```

- **Setup-синтаксис** (выше) — современный, привычный `ref/computed/function`.
- **Options-синтаксис**:

```ts
defineStore("cart", {
  state: () => ({ items: [] }),
  getters: {
    total: (s) => s.items.reduce((sum, i) => sum + i.price, 0),
    count: (s) => s.items.length,
  },
  actions: {
    async add(id: number) { this.items.push(await api.getProduct(id)); },
  },
});
```

### Что такое state/getters/actions

| Сущность | Аналог | Особенности |
|---|---|---|
| `state` | `data` | реактивные данные; `reset()` в Options-стиле |
| `getters` | `computed` | кэш, зависят от state/других getters |
| `actions` | методы | могут быть async, вызывать другие actions |

```ts
const cart = useCartStore();
cart.items;      // реактивно
cart.total;      // getter
await cart.add(1); // action
```

### Деструктуризация

- **Не** деструктурируйте без `storeToRefs`: `const { total } = cart` теряет реактивность (для getters/state).
- `const { items, total } = storeToRefs(cart)` — только state/getters; методы берите напрямую (`cart.add`).

### Обновление state

- Напрямую (`cart.items.push(...)`) — можно, Vue отследит.
- Через actions — точка входа для логики/валидации/побочек.
- `$patch` — батч-обновление (Options-стиль), `$reset()` — сброс (только Options-синтаксис).

### Подвох

- Два экземпляра стора `useCartStore()` — **один и тот же** синглтон на приложение (по id), а не копия.
- `state` возвращает прокси; сравнение с исходным объектом — по ссылке.
- В SSR (Nuxt) стор — **на один запрос**; глобальная переменная в модуле = утечка между пользователями.

## Связанные вопросы

- [[22 Мутировать state в Pinia]]
- [[23 Vuex mutations и actions]]
- [[24 Pinia и Vuex]]

## Ресурсы

- Pinia: https://pinia.vuejs.org/
