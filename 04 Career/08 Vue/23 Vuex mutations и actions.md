---
type: careers/interview
section: Vue
topic: State
status: ready
created: 2026-11-16
tags: [interview, vue, vuex]
---

# Vuex: mutations и actions

## Краткий ответ (30 секунд)

В Vuex `state` меняется **только синхронными `mutations`** (каждая — функция `(state, payload)`, вызывается через `commit`). **`actions`** — асинхронные операции: делают запросы, вызывают несколько мутаций (`dispatch`), могут возвращать промис. Смысл разделения — **трассируемость**: каждая мутация — точка изменения в devtools (time travel), а action — «команда» с побочными эффектами. В модульном варианте `namespaced: true` — пространства имён. Сейчас для новых проектов — Pinia (в ней действий/mutations разделения нет).

## Подробно

### Vuex 4 (Vue 3)

```js
const store = createStore({
  state: () => ({ count: 0 }),
  mutations: {
    increment(state, amount = 1) {
      state.count += amount; // только синхронно
    },
  },
  actions: {
    async incrementAsync({ commit }, amount) {
      await delay(500);
      commit("increment", amount);
    },
  },
  getters: {
    double: (state) => state.count * 2,
  },
});
```

```js
store.commit("increment", 2);          // sync
await store.dispatch("incrementAsync", 2); // async
```

### Почему такое разделение

- **Devtools**: список мутаций, time travel (откат состояния в точку).
- **Strict mode**: `strict: true` бросает ошибку при прямом изменении state вне мутации.
- **Побочные эффекты** (fetch) — не в мутациях, их повтор незаписываем.

### Модули

```js
const user = {
  namespaced: true,
  state: () => ({ profile: null }),
  mutations: {
    set(state, profile) { state.profile = profile; },
  },
  actions: {
    async load({ commit }) { commit("set", await api.me()); },
  },
};
```

- Вызов: `dispatch("user/load")`.
- В Pinia — просто отдельные сторы (без `namespaced`).

### Антипаттерны

- `this.state.x = ...` — прямое изменение (strict mode ловит).
- Fetch в mutation — mutation должна быть чистой и синхронной.
- Сотни действий «то же, что commit» — церемония без смысла (Pinia её убрала).

### Сравнение с Pinia

| | Vuex | Pinia |
|---|---|---|
| Изменение state | только mutation | напрямую/actions |
| Мутации | ✅ | ❌ |
| Actions | async + commit | async + прямые изменения |
| TS | сложнее | отлично |
| Модули | `namespaced` | отдельные сторы |
| Vue 2/3 | 3/4 | 3 |

### Миграция

`pinia` умеет работать рядом; для больших Vuex — постепенно, либо переход на `@vue/compat` для Vue 3. Многие проекты оставляют Vuex, если он работает.

> [!tip] Как отвечать
> «Mutation — синхронная точка изменения state для трассировки; action — async-операция с побочными эффектами, которая коммитит мутации. В Pinia этого деления нет: actions меняют state напрямую, а состояние реактивно».

## Связанные вопросы

- [[21 Store и Pinia]]
- [[24 Pinia и Vuex]]
- [[22 Мутировать state в Pinia]]

## Ресурсы

- Vuex: https://vuex.vuejs.org/
