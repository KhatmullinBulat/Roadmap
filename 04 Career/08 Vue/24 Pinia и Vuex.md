---
type: careers/interview
section: Vue
topic: State
status: ready
created: 2026-11-16
tags: [interview, vue, pinia, vuex]
---

# Pinia и Vuex

## Краткий ответ (30 секунд)

**Pinia** — официальный стор для Vue 3 (и Vue 2), построен на Composition API: стор — это `defineStore` с `state/getters/actions`, почти любой composable. Нет мутаций, типизация выводится, модули — просто отдельные сторы, есть `$patch`/`$subscribe`/devtools. **Vuex** — исторический стор с мутациями, actions, модулями (`namespaced`), сложной типизацией. В новых проектах выбирают Pinia: меньше церемоний, лучше TypeScript, лучше tree-shaking, проще тесты. Vuex в 4-й версии поддерживается, но развитие минимальное (команда рекомендует Pinia).

## Подробно

### Архитектурно

**Vuex** — «flux-подобный»: `state → view → action → mutation → state`. Централизованный `store` с `modules`.

**Pinia** — набор независимых сторов: `useCartStore()`, `useUserStore()`, каждый — модуль с замыканием. Композиция важнее наследования.

### Таблица

| | Pinia | Vuex 4 |
|---|---|---|
| API | setup-сторе (ref/computed/fn) или options | options/modules |
| Мутации | нет | обязательны |
| TypeScript | вывод из кода | GenModule, ручные типы |
| Модули | отдельные сторы | `modules` + `namespaced` |
| SSR | state на запрос | state на запрос |
| Devtools | ✅ | ✅ |
| `$reset` | Options-стиль | есть |
| Composition | `useX()` | mixins/мапперы |
| Поддержка | активная | поддержка, без нового |

### Пример «одно и то же»

Vuex:

```js
store.state.count;
store.commit("increment");
store.dispatch("load");
```

Pinia:

```js
const store = useCounter();
store.count;
store.increment();
await store.load();
```

### Почему Pinia

- **Меньше кода**: нет мутаций, нет `namespaced`, нет мапперов.
- **TS** «из коробки».
- **Композиция**: стор можно вызывать из другого стора/композабла.
- **Размер**: меньше.
- **Vue 3-нативность**; Vuex 4 поддерживает Vue 3, но с оговорками.

### Когда Vuex

- Огромный legacy на Vuex (миграция — отдельный проект).
- Требуется строгий time-travel/strict mode (в Pinia есть devtools, но не «мутации» как скачок).
- Команда привыкла и проект стабилен.

### Миграция

- Pinia и Vuex могут сосуществовать.
- Переносить по одному стору/модулю; компоненты переключать на `useXStore()`.
- В Nuxt — `@pinia/nuxt` (автоимпорт `useXStore`, SSR-friendly).

## Связанные вопросы

- [[21 Store и Pinia]]
- [[23 Vuex mutations и actions]]
- [[22 Мутировать state в Pinia]]

## Ресурсы

- Pinia vs Vuex: https://pinia.vuejs.org/introduction.html
