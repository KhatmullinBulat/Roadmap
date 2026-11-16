---
type: careers/interview
section: Vue
topic: Core
status: ready
created: 2026-11-16
tags: [interview, vue, composition-api]
---

# Composition API: setup, композаблы и Options API

## Краткий ответ (30 секунд)

**Options API** — компонент описывается «опциями»: `data`, `computed`, `methods`, `watch`, хуки. Логика одной фичи **размазана** по опциям, переиспользование — миксины (конфликты имён, неявные зависимости). **Composition API** — `setup`/`<script setup>`, где логика **сгруппирована по фиче**, состояние через `ref`/`reactive`, функции обычные; переиспользование — **композаблы** (`useX()`) с явными зависимостями и возвратом. Composition API не заменяет Options: в Vue 3 оба работают, а в одном проекте их можно смешивать (Options-компонент может использовать композаблы).

## Подробно

### Options API

```vue
<script>
export default {
  data: () => ({ search: "", results: [] }),
  computed: {
    hasResults() { return this.results.length > 0; },
  },
  watch: {
    search: { handler: "searchUsers", immediate: true },
  },
  methods: {
    async searchUsers() {
      this.results = await api.search(this.search);
    },
  },
  mounted() { this.searchUsers(); },
};
</script>
```

- Плюсы: понятно новичкам, `this`-контекст, строгая структура.
- Минусы: `this` (потеря контекста), логика по фичам разделена, миксины.

### Composition API

```vue
<script setup lang="ts">
import { ref, watch } from "vue";

const search = ref("");
const results = ref<Result[]>([]);

watch(search, async (query) => {
  results.value = await api.search(query);
}, { immediate: true });
</script>
```

- Функции — обычные, `this` не нужен.
- Логика рядом: search + результаты + обработчики.
- TS выводится.

### Композабл

```ts
// composables/useSearch.ts
export function useSearch(fetcher: (q: string) => Promise<Result[]>) {
  const query = ref("");
  const results = ref<Result[]>([]);
  const loading = ref(false);

  watch(query, async (q) => {
    loading.value = true;
    try { results.value = await fetcher(q); }
    finally { loading.value = false; }
  });

  const clear = () => { query.value = ""; results.value = []; };

  return { query, results, loading, clear };
}
```

- Явные вход/выход, нет `this`, легко тестировать.
- Хуки жизненного цикла внутри композабла привязываются к **текущему компоненту** (`onMounted`, `onUnmounted`) — автоматически (в `setup`).

### `setup` и `<script setup>`

- `setup(props, { emit, expose, attrs, slots })` — вызывается до `created`.
- Возврат: объект, доступный в шаблоне.
- `defineProps`/`defineEmits`/`defineExpose`/`defineModel` компилируются в `props`/`emit`.

### Миксины vs композаблы

| | Миксины | Композаблы |
|---|---|---|
| Источник данных | неявно `this` | явно параметры/возврат |
| Конфликты имён | да | нет |
| Отладка | плохо | стек явный |
| Типы | нет | отлично |
| Порядок | зависит от merge | явный |

### Когда Options всё ещё ок

- Небольшие компоненты/прототипы.
- Команда привыкла, legacy.
- Библиотеки с Options-контрактом (mixin-style).

### Смешивание

- Options-компонент может вызывать композабл в `setup()` (если он есть).
- `setup()` в Options должен возвращать объект для `this`/шаблона.

> [!tip] Как отвечать
> «Composition API — это про группировку логики и переиспользование. Композабл — функция с явным входом и `ref`/функциями на выходе; в отличие от миксинов, без конфликтов и с типами. Options API в Vue 3 остаётся, но для сложной логики — `script setup`».

## Связанные вопросы

- [[11 ref и reactive]]
- [[03 Хуки жизненного цикла]]
- [[01 Vue 2 и Vue 3]]

## Ресурсы

- Vue Composition API: https://vuejs.org/guide/extras/composition-api-faq
