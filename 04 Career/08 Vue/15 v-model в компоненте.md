---
type: careers/interview
section: Vue
topic: Компоненты
status: ready
created: 2026-11-16
tags: [interview, vue, v-model]
---

# v-model в компоненте

## Краткий ответ (30 секунд)

На **нативном элементе** `v-model` — сахар: для `input` — `:value` + `@input` (с `trim`/`number`/`lazy` модификаторами). На **компоненте** `v-model="x"` разворачивается в **`:model-value="x"` + `@update:model-value="x = $event"`**. Для нескольких значений — именованные: `v-model:title="title"` → prop `title` + событие `update:title`. С Vue 3.4 можно объявлять через **`defineModel()`** вместо ручного `defineProps`/`defineEmits`. Имя `modelValue`/`update:modelValue` — конвенция Vue 3 (в Vue 2 было `value`/`input` для `v-model`).

## Подробно

### Как выглядит

```vue
<!-- Parent -->
<MyInput v-model="name" />
```

Эквивалентно:

```vue
<MyInput :model-value="name" @update:model-value="name = $event" />
```

### Компонент (явно)

```vue
<!-- MyInput.vue -->
<script setup>
const props = defineProps<{ modelValue: string }>();
const emit = defineEmits<{ "update:modelValue": [value: string] }>();
</script>

<template>
  <input
    :value="props.modelValue"
    @input="emit('update:modelValue', ($event.target as HTMLInputElement).value)"
  />
</template>
```

### Компонент через `defineModel` (3.4+)

```vue
<script setup>
const model = defineModel<string>(); // modelValue + update:modelValue
const model2 = defineModel<number>("count"); // v-model:count
</script>

<template>
  <input v-model="model" />
</template>
```

- `defineModel` возвращает ref, читаемый/записываемый; родитель получает обновление.
- Можно `defineModel({ default: "", required: false })`.
- Внутри — `props` + `emit`, просто компактно.

### Несколько `v-model`

```vue
<UserForm v-model:first-name="first" v-model:last-name="last" />
```

Компонент:

```vue
<script setup>
const first = defineModel<string>("firstName");
const last = defineModel<string>("lastName");
</script>
```

### Модификаторы

```vue
<MyInput v-model.trim="email" v-model:age.number="age" />
```

```js
const [model, modifiers] = defineModel({ set: (v) => modifiers.capitalize ? capitalize(v) : v });
```

- Встроенные: `.lazy` (change вместо input), `.number`, `.trim` — для нативного `v-model`.
- Для компонента — `defineModel` даёт `modifiers`.

### Подвохи

```vue
<!-- ❌ мутация пропса вместо emit -->
<input :value="modelValue" @input="modelValue = ..." />
```

- Если компонент получает `modelValue`, но не емитит — родитель не узнает об изменении.
- Если `v-model` на компоненте, который не объявил `emits: ["update:modelValue"]`, Vue передаст событие как атрибут `onUpdate:modelValue` в корень (может «сработать» случайно) — обычно ломается у многослойных компонентов.
- `v-model` на `reactive`-поле — работает (`v-model="form.name"`), на `ref` — `v-model="name"` (без `.value` в шаблоне).

### `v-model` vs `value` + `@input` вручную

- `v-model` — конвенция и меньше кода.
- Ручная схема — прозрачнее, легче модифицировать (например, не обновлять состояние, пока валидация не пройдёт).
- См. [[16 v-model vs prop emit]].

> [!tip] Как отвечать
> «`v-model` — это sugar: prop `modelValue` + событие `update:modelValue`. Именованные — `v-model:name` → prop `name` + `update:name`. С 3.4 — `defineModel`. В Vue 2 конвенция была `value`/`input`».

## Связанные вопросы

- [[16 v-model vs prop emit]]
- [[17 emit и defineEmits]]
- [[05 Мутация props]]

## Ресурсы

- Vue v-model: https://vuejs.org/guide/components/v-model
