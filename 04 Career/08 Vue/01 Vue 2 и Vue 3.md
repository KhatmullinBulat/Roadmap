---
type: careers/interview
section: Vue
topic: Core
status: ready
created: 2026-11-16
tags: [interview, vue, vue2, vue3]
---

# Vue 2 vs Vue 3 на практике

## Краткий ответ (30 секунд)

**Vue 3** — новый движок: реактивность на **Proxy** (Vue 2 — `Object.defineProperty`), **Composition API** (логика по фичам вместо опций), **Fragments** (несколько корневых), `<script setup>`, **Teleport/Suspense**, лучший TypeScript, меньший бандл и быстрее. **Vue 2** — EOL с **31 декабря 2023** (последняя 2.7.16), получает только платный Extended LTS. На практике на Vue 2 остаются legacy-проекты: миграция через `@vue/compat` (Vue 3 в режиме совместимости). Options API **не исчез** — он работает в Vue 3; Composition API добавляет переиспользование и типизацию.

## Подробно

### Реактивность

| | Vue 2 | Vue 3 |
|---|---|---|
| Механизм | `Object.defineProperty` (геттеры/сеттеры) | `Proxy` |
| Новые ключи объекта | `Vue.set(obj, "new", 1)` | обычное `obj.new = 1` |
| Удаление ключа | `Vue.delete` | `delete obj.key` |
| Индексы массива | `arr[i] = x` не реактивно | реактивно |
| Производительность | ок, но «промах» по новым свойствам | ленивая, лучше память |
| Классовые/Map/Set | не отслеживаются | отслеживаются (Proxy) |
| `ref` для примитивов | нет | да |

```js
// Vue 2 — без Vue.set новый ключ не реактивен
this.$set(this.user, "email", "a@b.c");

// Vue 3 — просто работает
user.email = "a@b.c";
```

### API

**Options API** (`data`, `methods`, `computed`, `watch`, `mounted`) — всё ещё поддерживается и часто проще для мелких компонентов. **Composition API** (`setup`, `ref`, `computed`, `watch`) — лучше для сложной логики:

```vue
<script setup>
import { ref, computed } from "vue";

const count = ref(0);
const double = computed(() => count.value * 2);
</script>

<template>
  <button @click="count++">{{ count }} / {{ double }}</button>
</template>
```

- `<script setup>` — меньше бойлерплейта, лучший вывод типов.
- Логика группируется по фиче, а не по «опциям».
- `useXxx`-композаблы — переиспользование вместо миксинов.

### Шаблоны и рендеринг

- **Fragments**: несколько корневых узлов (Vue 2 — один корень).
- **Teleport**, **Suspense**, `v-model` на компоненте = `modelValue` + `update:modelValue`.
- **v-if** приоритет выше **v-for** (в Vue 3), в Vue 2 наоборот — было частой ошибкой.
- `<script setup>`-макросы: `defineProps`, `defineEmits`, `defineExpose`, `withDefaults`.

### Экосистема

- **Vuex** → **Pinia** (официальный стор с Vue 3).
- **Vue Router 3 → 4**: `createRouter`, `createWebHistory`, route props.
- **Vite** вместо Vue CLI (webpack).
- **Nuxt 3/4** вместо Nuxt 2 (на Vue 2).
- **Element UI, Vuetify 2** — legacy; современные — Element Plus, Vuetify 3.

### Поддержка

- Vue 2 — **EOL 31.12.2023**, только критичные/безопасность → платный Extended LTS.
- Vue 3 — текущая версия; **3.6 (Vapor Mode)** на момент 2026 — RC, Vapor — opt-in.
- Миграция: сначала всё на 3 через `@vue/compat`, потом постепенно убрать deprecated.

### Что ответить на «чем на практике»

> «На Vue 2 я использовал бы `Vue.set` при добавлении полей и миксины для общей логики. Во Vue 3 — `script setup`, композаблы, Pinia и Vite. Заодно исчезли ограничения: несколько корней, реактивные новые ключи, нормальный TS».

## Связанные вопросы

- [[10 Как работает реактивность Vue 3]]
- [[03 Хуки жизненного цикла]]
- [[24 Pinia и Vuex]]

## Ресурсы

- Vue 3 migration guide: https://v3-migration.vuejs.org/
- Vue 2 EOL: https://v2.vuejs.org/eol/
