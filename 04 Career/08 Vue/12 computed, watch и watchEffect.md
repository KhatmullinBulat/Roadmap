---
type: careers/interview
section: Vue
topic: Реактивность
status: ready
created: 2026-11-16
tags: [interview, vue, computed, watch]
---

# computed, watch и watchEffect

## Краткий ответ (30 секунд)

**`computed`** — **производное значение** с кэшем: пересчитывается лениво, только когда изменились зависимости; возвращает ref, вычисление чистое (без побочных эффектов). **`watch`** — **реакция на изменение** явно указанного источника (ref, getter, массив), для побочных эффектов (запрос, лог, localStorage), с `deep`, `immediate`, `flush`, `once` (3.4+). **`watchEffect`** — автоматически собирает зависимости из тела, удобен, когда зависимости очевидны; менее контролируем. Правило: значение → `computed`; действие → `watch`; «пробежаться по всем используемым» → `watchEffect`.

## Подробно

### `computed`

```js
const firstName = ref("Анна");
const lastName = ref("Иванова");
const fullName = computed(() => `${firstName.value} ${lastName.value}`);
```

- Кэш: если зависимости не менялись — значение берётся из кэша.
- Ленивый: не считается, пока не прочитан.
- Только геттер — **нельзя** присваивать (`fullName.value = ...` → предупреждение). Для записи — `computed({ get, set })`.
- В `computed` **не делать** side effects (fetch, мутации).
- Оптимизирует шаблон: `{{ filteredItems.length }}` вместо метода, вызываемого на каждый ререндер.

### `watch`

```js
watch(
  () => user.value.id,
  async (id, oldId) => {
    if (!id) return;
    data.value = await fetchUser(id);
  },
  { immediate: true },
);
```

- Источник: ref, getter (чтобы реагировать на поля), массив источников.
- Колбэк `(newVal, oldVal)`.
- **`deep`** — глубокая проверка; по умолчанию для `reactive` объектов — неглубоко/по ссылке; для коллекций — `deep: true`.
- **`immediate`** — вызвать сразу.
- **`flush: "pre"|"post"|"sync"`** — до/после DOM-обновления (по умолчанию `pre`).
- **`once: true`** — сработать один раз (Vue 3.4+).
- Cleanup: `onCleanup` в колбэке — отменять предыдущий запрос.

```js
watch(query, (q, _, onCleanup) => {
  const ctrl = new AbortController();
  onCleanup(() => ctrl.abort());
  search(q, { signal: ctrl.signal });
});
```

### `watchEffect`

```js
watchEffect(async (onCleanup) => {
  const ctrl = new AbortController();
  onCleanup(() => ctrl.abort());
  data.value = await search(query.value, { signal: ctrl.signal });
});
```

- Зависимости **автоматически** собираются из тела (как `computed`).
- Запускается сразу (immediate по умолчанию) и при изменениях.
- **Минус**: легче поймать «слишком много зависимостей» (случайное чтение) и сложнее отладить «почему сработало». Не читайте `.value` в условии, если не хотите подписки (см. `toRef`).
- `watch` предпочтительнее, когда важно контролировать источник.

### Сравнение

| | `computed` | `watch` | `watchEffect` |
|---|---|---|---|
| Возвращает | значение (ref) | — (только побочный эффект) | — |
| Запуск | лениво, при чтении | при изменении | сразу + при изменении |
| Зависимости | автоматически | явно указаны | автоматически (все чтения) |
| Кэш | ✅ | — | — |
| Побочные эффекты | ❌ | ✅ | ✅ |
| Контроль | высокий | высокий | средний |

### Когда что (практика)

- **computed**: `fullName`, `totalPrice`, `filteredList`, `isValid`.
- **watch**: «загрузить данные при смене `userId`», «сохранить в localStorage», «показать уведомление при ошибке», debounce-поиск.
- **watchEffect**: синхронизация простой анимации/размера, где все зависимости очевидны; логирование; проще на старте, но осторожно.

### Антипаттерны

- `computed` с `await fetch` — не работает и не нужно.
- `watch` на весь объект `reactive` без `deep` — не сработает на вложенные поля; с `deep` — может срабатывать слишком часто.
- `watch` + `watchEffect` для одного и того же.
- Мутация состояния в `watch` без защиты от циклов (A → B → A).

### `watch` и `props`

```js
const props = defineProps<{ id: number }>();
watch(() => props.id, (id) => load(id), { immediate: true });
```

Не забывайте `() => props.id`: `watch(props.id, ...)` не работает для примитивного пропса.

> [!tip] Как отвечать
> «`computed` — для значений, он кэширует. `watch` — для действий с явными источниками и контролем. `watchEffect` — когда зависимости очевидны и нужен автосбор; но тогда риск лишних зависимостей».

## Связанные вопросы

- [[10 Как работает реактивность Vue 3]]
- [[11 ref и reactive]]
- [[13 toRefs и деструктуризация]]

## Ресурсы

- Vue computed: https://vuejs.org/guide/essentials/computed
- Vue watchers: https://vuejs.org/guide/essentials/watchers
