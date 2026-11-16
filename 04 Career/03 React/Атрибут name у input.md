---
type: careers/interview
section: React
topic: Формы
status: ready
created: 2026-11-16
tags: [interview, react, forms, html]
---

# Атрибут name у input

## Краткий ответ (30 секунд)

`name` — имя поля, под которым его значение отправляется на сервер в **`FormData`** (и автоматически становится ключом в **URL-encoded**, если формы нет иначе). В React он важен: 1) **`form.elements.namedItem("name")`** и `new FormData(form)` смотрят по `name`; 2) **uncontrolled-формы** (`defaultValue`/`defaultChecked`) получают значение по `name`; 3) `react-hook-form` регистрирует поля по `name`; 4) в **React 19 `<form action={fn}>`** функция получает `FormData`, и без `name` поле не отправится. `id` — это для `label[for]`, `name` — для данных. Путать их — частая причина «поле не приходит».

## Подробно

### Что даёт `name`

```html
<form>
  <label for="email">Email</label>
  <input id="email" name="email" type="email" />
  <input name="tags" value="a,b" />
  <select name="role"><option value="admin">admin</option></select>
  <button name="action" value="save">Сохранить</button>
</form>
```

```js
const fd = new FormData(form);
fd.get("email"); // "anna@example.com"
fd.getAll("tags"); // ["a,b"] — или несколько чекбоксов с одним name
```

- **Без `name`** поле **не попадает** в `FormData`. Это самая частая ошибка.
- **Чекбоксы**: без `value` — `"on"`; с одним `name` — все отмеченные.
- **Кнопка с `name`/`value`** добавляет своё значение в `FormData`, если именно она инициировала отправку.
- **`multiple`** (select) — `getAll`.
- Файлы — `fd.get("avatar")` вернёт `File`, если у `input` есть `name`.

### `name` в React

**Неконтролируемые формы:**

```jsx
<form onSubmit={(e) => {
  e.preventDefault();
  const fd = new FormData(e.currentTarget, e.nativeEvent.submitter);
  console.log(Object.fromEntries(fd));
}}>
  <input name="title" defaultValue="" />
  <button type="submit">ОК</button>
</form>
```

Второй аргумент `FormData(form, submitter)` (поддержка в браузерах) включает кнопку-отправитель с её `name`/`value`.

**React 19 actions:**

```jsx
async function createPost(formData) {
  "use server";
  const title = formData.get("title");
}

<form action={createPost}>
  <input name="title" />
</form>
```

**React Hook Form:** `register("title")` использует `name` для связи поля с формой.

### `id` vs `name` vs `htmlFor`

| Атрибут | Для чего |
|---|---|
| `name` | ключ при отправке формы / регистрации поля |
| `id` | уникальный идентификатор в DOM, `label htmlFor`, `useId()` для SSR |
| `htmlFor` | связь label с input (в JSX) |
| `aria-*` | доступность |

`<label>` может обернуть `<input>` без `htmlFor` — тогда связь неявная. В React `for` пишется как `htmlFor`.

### Связанные ошибки

- **`name` не указан** → `FormData` пустой, сервер не получает значение.
- **`name` с пробелом/точкой** — не ошибка, но хуже для сериализации.
- **`name` продублирован у radio**: именно так объединяются радио-кнопки в группу — это нормально.
- **`value` без `onChange`** (контролируемое поле без обработчика) — read-only и warning React. В uncontrolled используйте `defaultValue`.

### `useId` — для чего

```jsx
const id = useId();
return (
  <>
    <label htmlFor={id}>Имя</label>
    <input id={id} name="name" />
  </>
);
```

`useId` даёт одинаковый id на сервере и клиенте (гидрация). Но `name` всё равно пишется вручную.

> [!tip] Как отвечать
> «`name` — ключ поля в `FormData`. В React это основа и неуправляемых форм, и `register`, и React 19 actions. Без `name` значение не отправится. `id`/`htmlFor` — про label и доступность, не про данные».

## Связанные вопросы

- [[Библиотеки для форм RHF и RFF]]
- [[Что нового в React 18 и 19]]
- [[Всплытие и погружение событий]] — `submit`/делегирование

## Ресурсы

- MDN FormData: https://developer.mozilla.org/ru/docs/Web/API/FormData
- react.dev `<form>`: https://react.dev/reference/react-dom/components/form
