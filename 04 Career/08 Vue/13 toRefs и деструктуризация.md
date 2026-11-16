---
type: careers/interview
section: Vue
topic: Реактивность
status: ready
created: 2026-11-16
tags: [interview, vue, toRefs]
---

# toRefs и деструктуризация

## Краткий ответ (30 секунд)

`toRefs(reactiveObj)` превращает **каждое свойство** реактивного объекта в `ref`, сохраняя связь с оригиналом. Зачем: **деструктуризация** `reactive`-объекта ломает реактивность (`const { count } = state` копирует число), а `const { count } = toRefs(state)` даёт реактивный `count` (`count.value`). `toRef(obj, "key")` — тот же ref для одного свойства, **без** создания прокси. Типичные места: возврат из композабла/`setup` (потребитель может деструктурировать), `props` (не разрушать `props`), несколько `v-model`.

## Подробно

### Проблема

```js
const state = reactive({ count: 0, name: "Аня" });

const { count } = state;     // ❌ count — обычное число 0
count++;                     // меняет только локальную переменную
console.log(state.count);    // 0
```

### Решение

```js
import { toRefs } from "vue";

const state = reactive({ count: 0, name: "Аня" });
const { count, name } = toRefs(state);

count.value++;               // state.count === 1
```

- `toRefs` создаёт ref с getter/setter на исходный объект.
- В шаблоне `count` разворачивается автоматически.
- Деструктуризация в **шаблоне** не нужна — `state.count` работает и так.

### `toRef`

```js
const count = toRef(state, "count"); // один ref
```

- Отличие: `toRef` не обходит объект и не создаёт прокси; удобно, когда нужен один ключ.
- Если ключ не существует — ref создастся и будет писать в оригинал (но не реактивен до первого существующего чтения? — работает для `reactive`).

### Типовой кейс — композабл

```js
// useCounter.js
export function useCounter() {
  const state = reactive({ count: 0, step: 1 });
  const increment = () => (state.count += state.step);

  return {
    ...toRefs(state), // ✅ потребитель может деструктурировать
    increment,
  };
}
```

```js
const { count, increment } = useCounter();
count.value++; // реактивно
```

Без `toRefs` — `const { count } = useCounter()` получил бы «оторванное» значение.

### Пропсы

```js
const props = defineProps<{ title: string }>();

// ❌ не деструктурируйте props (в Vue < 3.5 — реактивность теряется; в 3.5+ — реактивная деструктуризация через компилятор)
const { title } = props;

// ✅
const title = toRef(props, "title");
```

- В Vue 3.5+ `<script setup>` поддерживает **реактивную деструктуризацию** пропсов при компиляции (`const { title } = defineProps<...>()`), но `toRef` остаётся явным и совместимым.
- Для передачи «остатка» — `useAttrs()`/`$attrs`.

### `toRefs` и `v-model`

```vue
<Child v-model:first-name="firstName" v-model:last-name="lastName" />
```

Если данные в `reactive`, `toRefs` даёт `ref`, который можно биндить.

### `toValue` / `unref` (3.3+)

```js
toValue(refOrValue) // разворачивает, если ref
```

Полезно в функциях, принимающих `MaybeRef`.

### Ловушки

- `toRefs` не делает глубокую реструктуризацию: вложенные объекты остаются реактивными по ссылке.
- `toRefs` на `ref` (не reactive) — ошибка, нужен `toRef`.
- Если объект пересоздаётся (`state = reactive({})`), `toRefs`-ссылки остаются на старом объекте — не пересоздавайте.
- `toRefs` в `watch` — используйте `toRef`/getter, не весь объект.

> [!tip] Как отвечать
> «`toRefs` — чтобы отдать наружу объект, который деструктурируют, и не потерять реактивность. Композабл возвращает `...toRefs(state)`. Для одного свойства — `toRef`. Не деструктурировать `reactive` и `props` вручную».

## Связанные вопросы

- [[11 ref и reactive]]
- [[12 computed, watch и watchEffect]]
- [[14 Лишние обновления во Vue]]

## Ресурсы

- Vue toRefs: https://vuejs.org/api/reactivity-utilities#torefs
