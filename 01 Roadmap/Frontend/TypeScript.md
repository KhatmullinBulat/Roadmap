---
type: roadmap
category: frontend
status: planned
priority: high
created: 2026-07-27
tags: [typescript, roadmap]
---

# TypeScript

> TypeScript — надмножество JavaScript с статической типизацией, которое компилируется в обычный JS. Цель этого roadmap — уверенно типизировать прикладной код, разбираться в выводе типов и не превращать типы в «костыли».

# Что нужно знать

## Основы языка

- [ ] Что такое TypeScript и зачем он нужен
- [ ] Компиляция: `tsc`, `tsconfig.json`, `strict`
- [ ] Аннотации типов и вывод типов
- [ ] Примитивы: `string`, `number`, `boolean`, `null`, `undefined`, `void`, `never`, `unknown`, `any`
- [ ] Массивы, кортежи (tuple), `readonly`
- [ ] Объектные типы, `type` и `interface`
- [ ] Функции: параметры, возвращаемые значения, перегрузки
- [ ] Объединения (`|`), пересечения (`&`)
- [ ] Литеральные типы и `as const`
- [ ] Опциональные свойства и `?.`/`??` в типах

## Работа с типами

- [ ] Узкое сужение типов (narrowing) и проверки
- [ ] Type guards: `typeof`, `in`, `instanceof`, пользовательские предикаты (`value is T`)
- [ ] Дженерики: функции, классы, ограничения (`extends`)
- [ ] Ключевое слово `keyof`, `typeof` для типов
- [ ] Условные типы и `infer`
- [ ] Отображённые типы (mapped types)
- [ ] Утилитарные типы: `Partial`, `Required`, `Pick`, `Omit`, `Record`, `ReturnType`, `Parameters`, `Awaited`
- [ ] `satisfies` и `as` — когда что применять
- [ ] Структурная типизация и совместимость

## Классы и ООП

- [ ] Модификаторы `public`, `private`, `protected`, `readonly`
- [ ] `abstract` классы
- [ ] Реализация интерфейсов
- [ ] Декораторы (обзор)

## Модули и экосистема

- [ ] `import type` / `export type`
- [ ] Работа с `.d.ts` и сторонними библиотеками
- [ ] `@types/*` и `DefinitelyTyped`
- [ ] Настройка `tsconfig` под проект: `module`, `target`, `moduleResolution`, `paths`
- [ ] Сборка: `tsc`, Vite, esbuild, SWC; зачем нужен `type-check` отдельно
- [ ] Линтер: `typescript-eslint` и правила, полезные с типами

## Прикладные темы

- [ ] Типизация API-ответов и валидация на границе (zod, valibot)
- [ ] React: пропсы, `useState<T>`, `useReducer`, обработчики событий, `ReactNode`
- [ ] Node.js: типы `process`, `Buffer`, `fs`
- [ ] Работа с `Promise<T>` и `async`/`await` в типах
- [ ] Типизация `JSON.parse` — почему результат `any`

# Практика

- [ ] Перевести небольшой JS-проект на TypeScript с `strict: true`
- [ ] Написать типизированный клиент API без `any`
- [ ] Реализовать дженерик-функцию `groupBy<T, K>(items: T[], key: (item: T) => K)`
- [ ] Описать тип состояния формы и reducer для него
- [ ] Написать собственный type guard и предикат для discriminated union
- [ ] Разобрать ошибку из `strictNullChecks` и объяснить, почему компилятор её выдал
- [ ] Сравнить `unknown` и `any` на примере безопасного разбора ответа

# Связанные знания

- [[JavaScript]] — основа, без которой TypeScript не имеет смысла
- [[1. Типы данных]]
- [[2. Оператор typeof]] — `typeof` работает как в JS, так и в type guards
- [[2. Явное]] — приведение типов в JS в сравнении с проверками типов в TS

# Ресурсы

Документация:
https://www.typescriptlang.org/docs/handbook/intro.html
https://www.typescriptlang.org/tsconfig

Статьи:
https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
https://www.typescriptlang.org/docs/handbook/2/narrowing.html
https://www.typescriptlang.org/docs/handbook/2/generics.html
https://github.com/typescript-cheatsheets/react

Книги и курсы:
- «Programming TypeScript» — Boris Cherny
- «Effect TypeScript» — Boris Cherny (обновлённая версия)
- TypeScript Handbook (официальный, читается как книга)
