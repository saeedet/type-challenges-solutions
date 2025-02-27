---
id: 527
title: Append to Object
lang: ru
level: medium
tags: object-keys
---

## Проблема

Реализовать тип, который добавляет новое свойство к объекту.
Тип принимает три параметра, а результатом должен быть новый объект, в котором присутствует новое свойство.
Например:

```typescript
type Test = { id: "1" };
type Result = AppendToObject<Test, "value", 4>; // expected to be { id: '1', value: 4 }
```

## Решение

Когда мы работаем с объектами в TypeScript, часто на помощь приходят пересечения типов.
Эта проблема не является исключением из правил.
Пробуем реализовать тип, который принимает входной объект и делает пересечение с новым объектом, в котором находится искомое свойство.

```typescript
type AppendToObject<T, U, V> = T & { [P in U]: V };
```

Я надеялся, но такое решение не устроило тесты.
В тестах, автор проблемы ожидает плоский тип, а не пересечение типов.
Поэтому, нам нужно вернуть новый объект со всеми свойствами из входного, плюс само свойство, которое нужно добавить.
Без пересечений, в одном объекте.

Начнём с сопоставляющих типов и вернём объект со свойствами из входного объекта:

```typescript
type AppendToObject<T, U, V> = { [P in keyof T]: T[P] };
```

Теперь, добавим новое свойство из `U` к свойствам в `T`.
Для этого воспользуемся хитрым трюком.
TypeScript не запрещает передавать оператору `in` объединения типов.

```typescript
type AppendToObject<T, U, V> = { [P in keyof T | U]: T[P] };
```

С таким трюком, получаем свойства из объекта `T` и свойство из `U`.
Как раз то, что нам нужно.
Добавим ограничение, что `U` должно быть строкой, так как ключами объекта могут быть только строки.

```typescript
type AppendToObject<T, U extends string, V> = { [P in keyof T | U]: T[P] };
```

Единственное, что осталось починить, это ошибку компиляции.
TypeScript не гарантирует что обращение `T[P]` будет правильным, так как `P` это объединение свойств из `T` и `U`.
Следовательно, может быть ситуация, когда `T[P]` попытается обратиться к несуществующему свойству из `T`.

С помощью условных типов добавляем проверку на наличие свойства.
Если элемент из `P` присутствует в свойствах `T`, обращаемся к `T`, иначе, типом значения будет тип, который мы добавляем к объекту.

```typescript
type AppendToObject<T, U extends string, V> = {
  [P in keyof T | U]: P extends keyof T ? T[P] : V;
};
```

## Что почитать

- [Сопоставляющие типы](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
- [Условные типы](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
- [Объединения типов](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types)
