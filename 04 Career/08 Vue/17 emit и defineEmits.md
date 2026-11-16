---
type: careers/interview
section: Vue
topic: Компоненты
status: ready
created: 2026-11-16
tags: [interview, vue, emit, defineEmits]
---

# emit и defineEmits

## Краткий ответ (30 секунд)

`emit` — способ отправить **событие вверх** от ребёнка к родителю: ребёнок не меняет данные родителя, а сообщает «что-то произошло» с payload. В Composition API объявляют `const emit = defineEmits<{ save: [payload: Payload]; cancel: [] }>()` (или массив `["save", "cancel"]`); вызов — `emit("save", payload)`; родитель слушает `@save="handler"`. События **не всплывают** через компоненты автоматически, но `v-on` с атрибутным наследованием может попасть на корневой узел — поэтому объявляйте `emits` явно, чтобы Vue не считал событие «лишним атрибутом». В Options API — опция `emits` + `this.$emit`.

## Подробно

### Объявление и вызов

```vue
<script setup lang="ts">
type Payload = { id: number; title: string };

const emit = defineEmits<{
  save: [payload: Payload];
  cancel: [];
  "update:modelValue": [value: string];
}>();

emit("save", { id: 1, title: "Купить" });
emit("cancel");
</script>
```

- `defineEmits<{ events: [args] }>()` — типизированный TS-вариант с кортежем аргументов.
- `const emit = defineEmits(["save", "cancel"])` — без типов.
- **Не** деструктурируйте `emit` из `$emit` — только `defineEmits` в `script setup`.

### Родитель

```vue
<Child @save="onSave" @cancel="onCancel" />
```

```js
function onSave(payload) { ... }
```

- `@save="onSave"` — передаётся payload.
- `@save="(p) => save(p)"` — inline.
- Модификаторы `.once` у `v-on` для компонентов не работают как у DOM — используйте логику.

### Типизация (варианты)

```ts
// 1. named tuple
const emit = defineEmits<{ change: [id: number] }>();

// 2. call signatures (старый синтаксис)
const emit = defineEmits<{
  (e: "change", id: number): void;
  (e: "close"): void;
}>();
```

### Что не так с `$emit` в Options API

```js
export default {
  emits: ["save"],
  methods: {
    handle() { this.$emit("save", { id: 1 }); },
  },
};
```

- Без `emits` Vue добавит `onSave` как атрибут на корневой DOM — и событие **всплывёт** как DOM-событие.
- `emits` — «декларация контракта», также влияет на `$attrs`.

### `v-model` и `emit`

`defineModel()` — синтаксический сахар для `props` + `emit("update:modelValue")`. Если нужен `v-model`, событие должно называться `update:modelValue` (или `update:<name>`).

### Антипаттерны

- Отправлять объект состояния целиком («просто отдайте родителю») — лучше событие-намерение + минимум данных.
- `emit` в `computed`/`watch` без необходимости — событие должно быть результатом действия.
- Имена событий без действия (`data`, `value`) — лучше `save`, `select`, `open`, `update:modelValue`.
- Многоуровневая цепочка emit-ов через 3 компонента — сигнал: состояние/событие нужно поднять выше или в стор.

### Тестирование

```js
const wrapper = mount(Child);
await wrapper.find("button").trigger("click");
expect(wrapper.emitted("save")![0][0]).toEqual({ id: 1 });
```

> [!tip] Как отвечать
> «`emit` — контракт вверх, объявляем `defineEmits` с типами. Родитель слушает `@event`. Без объявления событие становится атрибутом DOM и может всплыть — это частый баг. `v-model` — это `update:modelValue`».

## Связанные вопросы

- [[15 v-model в компоненте]]
- [[16 v-model vs prop emit]]
- [[05 Мутация props]]

## Ресурсы

- Vue events: https://vuejs.org/guide/components/events
- `defineEmits`: https://vuejs.org/api/sfc-script-setup#defineemits
