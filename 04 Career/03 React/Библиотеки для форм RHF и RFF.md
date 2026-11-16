---
type: careers/interview
section: React
topic: Формы
status: ready
created: 2026-11-16
tags: [interview, react, forms]
---

# Библиотеки для форм: RHF и RFF

## Краткий ответ (30 секунд)

**RHF (React Hook Form)** — форма на **неконтролируемых** полях: регистрирует входы через `ref`, хранит значения вне React-состояния и ререндерит только то, что подписано (`useWatch`), поэтому быстрая на больших формах. **RFF (React Final Form)** — обёртка над **Final Form**: форма на подписках и «рендер-пропсах», сознательно ориентирована на поля в едином источнике истины и минимальные ререндеры через `Field`/`FormSpy`. Обе поддерживают валидацию, но RHF сейчас более распространён; RFF — зрелый, но менее активный. Третий частый вариант — **Formik** (проще, но контролируемый, медленнее на больших формах) и нативный `<form action>` с React 19.

## Подробно

### React Hook Form (RHF)

```jsx
import { useForm } from "react-hook-form";

function Login() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm({ defaultValues: { email: "", password: "" } });

  const onSubmit = async (values) => {
    await api.login(values);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <input {...register("email", { required: "Введите email" })} />
      {errors.email && <p>{errors.email.message}</p>}

      <input type="password" {...register("password", { minLength: 8 })} />
      {errors.password && <p>{errors.password.message}</p>}

      <button disabled={isSubmitting}>Войти</button>
    </form>
  );
}
```

- **Неконтролируемые входы** + `ref` — минимум рендеров.
- **`register`** возвращает `name`, `ref`, `onChange`, `onBlur` — поля знают форму.
- **`useWatch`/`watch`** — подписка на конкретные поля (только они ререндерятся).
- **Валидация**: `resolver` (zod/yup), схемы в `zodResolver`.
- **`formState.isSubmitting/isValid/isDirty`** — состояния формы.
- **`Controller`** — для контролируемых компонентов (UI-кит, `react-select`).
- **`useFieldArray`** — динамические списки полей.
- **`FormProvider`** — проброс формы через контекст.
- **`setError`** — серверные ошибки.

Плюсы: скорость на больших формах, меньше кода, хорошая TS-поддержка. Минусы: «магия» с refs, не всё «из коробки» для сложного UX, иногда отладка не так очевидна.

### React Final Form (RFF)

```jsx
import { Form, Field } from "react-final-form";

<Form
  onSubmit={async (values) => { await api.save(values); }}
  validate={(values) => {
    const errors = {};
    if (!values.email) errors.email = "Введите email";
    return errors;
  }}
  render={({ handleSubmit, submitting, form }) => (
    <form onSubmit={handleSubmit}>
      <Field name="email">
        {({ input, meta }) => (
          <>
            <input {...input} type="email" />
            {meta.touched && meta.error && <p>{meta.error}</p>}
          </>
        )}
      </Field>
      <button type="submit" disabled={submitting}>Сохранить</button>
    </form>
  )}
/>
```

- Построен на **Final Form** — библиотека подписок.
- **`Field`** — рендер-пропсы для поля; ререндер только подписанного поля.
- **`FieldArray`**, **`FormSpy`** — массивы и «шпион» за формой.
- `validate`/`validateOnBlur`, `subscription` — тонкий контроль рендеров.
- `react-final-form-hooks` — хуки.
- Плюсы: предсказуемая архитектура, гибкие рендер-пропсы, без контролируемого поля в state.
- Минусы: **активность проекта** ниже (последние релизы редки), экосистема UI-интеграций меньше, рендер-пропсы многословны.

### Сравнение

| | RHF | RFF | Formik |
|---|---|---|---|
| Модель | неконтролируемая | подписки/Final Form | контролируемая |
| Ререндеры | минимум | минимум | много на больших формах |
| API | `register`/`Controller` | `Field`/рендер-пропсы | `Formik`/`Field` |
| Активность | высокая | низкая | средняя |
| ТС | хорошая | хорошая | средняя |
| Валидация | resolver (zod/yup) | `validate` синхронный/`parse` | `validate`/`yup` |

### Когда библиотека не нужна

- Форма из 2–3 полей и нативный `FormData` → `useState` или **`<form action={fn}>` (React 19)**: `action` получает `FormData`, `useActionState` даёт pending/ошибки, `useOptimistic` — оптимистичное состояние.

```jsx
const [error, submit, isPending] = useActionState(async (prev, formData) => {
  const res = await api.save(Object.fromEntries(formData));
  return res.error ?? null;
}, null);

<form action={submit}>
  <input name="email" />
  {error && <p>{error}</p>}
</form>
```

## Связанные вопросы

- [[Атрибут name у input]]
- [[Что нового в React 18 и 19]]
- [[Хуки React — обзор]]

## Ресурсы

- RHF: https://react-hook-form.com/
- React Final Form: https://final-form.org/react
