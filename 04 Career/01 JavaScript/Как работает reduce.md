---
type: careers/interview
section: JavaScript
topic: Массивы
status: ready
created: 2026-11-16
tags: [interview, javascript, arrays, reduce]
---

# Как работает reduce

## Краткий ответ (30 секунд)

`reduce(callback, initialValue)` последовательно «сворачивает» массив в одно значение. На каждом шаге вызывается `callback(acc, currentValue, index, array)`, где `acc` — накопленный результат предыдущего шага, а возвращённое значение становится новым `acc`. Если `initialValue` задан, первый вызов идёт с ним и с `arr[0]`; **если не задан** — `acc = arr[0]`, обход начинается с `arr[1]`. На пустом массиве без начального значения `reduce` бросает `TypeError`.

## Подробно

### Порядок вызовов

```js
const sum = [1, 2, 3].reduce((acc, cur) => acc + cur, 0);

// шаги:
// acc=0, cur=1 → 1
// acc=1, cur=2 → 3
// acc=3, cur=3 → 6
// результат: 6
```

Без `initialValue`:

```js
[1, 2, 3].reduce((acc, cur) => acc + cur);
// acc=1, cur=2 → 3
// acc=3, cur=3 → 6
```

Разница видна на строках и объектах:

```js
[].reduce((a, b) => a + b);         // TypeError: Reduce of empty array with no initial value
[].reduce((a, b) => a + b, 0);      // 0

["a", "b"].reduce((a, b) => a + b); // "ab"
["a"].reduce((a, b) => a + b);      // "a" (вызовов callback не было)
```

### Что можно сделать (и этого почти всегда достаточно)

```js
// 1. Сумма/произведение
[1, 2, 3].reduce((s, x) => s + x, 0);

// 2. Группировка
const byRole = users.reduce((acc, u) => {
  (acc[u.role] ??= []).push(u);
  return acc;
}, {});

// 3. Индексация по id
const byId = items.reduce((acc, item) => {
  acc[item.id] = item;
  return acc;
}, {});

// 4. Подсчёт частот
const freq = words.reduce((acc, w) => {
  acc[w] = (acc[w] ?? 0) + 1;
  return acc;
}, {});

// 5. Композиция функций
const pipe = (...fns) => (x) => fns.reduce((v, f) => f(v), x);
```

### Сигнатура

```js
arr.reduce((acc, currentValue, index, array) => nextAcc, initialValue);
```

`index` — индекс текущего элемента; `array` — исходный массив. Параметры после `acc`/`currentValue` нужны редко, но их надо знать.

### Подвохи

1. **Пустой массив:** `[].reduce(fn)` — `TypeError`, пустая строка без initial тоже.  
2. **`initialValue = undefined`:** это *заданное* значение, но передавать `undefined` явно — почти всегда ошибка: яснее `0`, `""` или `[]`.  
3. **Мутация `acc`:** если `acc` — исходный объект, `reduce` вернёт тот же объект. Для иммутабельности возвращайте новый (`{...acc, [k]: v}`), а не мутируйте.  
4. **Разреженные массивы:** `reduce` пропускает «дырки» (не как `fill`).  
5. **Не заменяет `forEach`:** если побочный эффект без накопления — используйте `forEach`/`for...of`.

### Когда не использовать `reduce`

- Если это `map` или `filter` — читаемее и быстрее специальные методы.
- Если тело многократно вложено (`reduce` внутри `reduce`) — императивный `for` читается лучше.
- Для простых сумм — тот же `for` в 3 раза понятнее новичку.

> [!tip] Как отвечать
> «`reduce` — свёртка массива. `acc` — аккумулятор, если нет `initialValue`, им становится первый элемент. На пустом массиве без initial — `TypeError`. Классика — группы по ключу и индексация по id».

## Связанные вопросы

- [[Мутирующие и немутирующие методы массива]]
- [[Каррирование]] — `pipe`/`compose`
- [[Map и объект]]

## Ресурсы

- MDN: https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
