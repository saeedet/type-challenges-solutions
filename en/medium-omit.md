<h1>Omit <img src="https://img.shields.io/badge/-medium-eaa648" alt="medium"/> <img src="https://img.shields.io/badge/-%23union-999" alt="#union"/> <img src="https://img.shields.io/badge/-%23built--in-999" alt="#built-in"/></h1>

## Challenge

Implement the built-in `Omit<T, K>` generic without using it.

Constructs a type by picking all properties from `T` and then removing `K`

For example

```ts
interface Todo {
  title: string
  description: string
  completed: boolean
}

type TodoPreview = MyOmit<Todo, 'description' | 'title'>

const todo: TodoPreview = {
  completed: false,
}
```

## Solution

We need to return a new object type here, but without specified keys.
Obviously, it is a flag that we need to use [mapped types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types) here.
We need to map each property in object and construct a new type.

Let us start with the basic block and construct the same object:

```ts
type MyOmit<T, K> = { [P in keyof T]: T[P] }
```

Here, we iterate over all the keys in T, map it to the type P and make it a key in our new object type, while value type is the type from T[P].

That way, we iterate over all the keys, but we need to filter out those that we are not interested in.
To achieve that, we can [remap the key type using “as” syntax](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-1.html#key-remapping-in-mapped-types):

```ts
type MyOmit<T, K> = { [P in keyof T as P extends K ? never : P]: T[P] }
```

We map all the properties from T and if the property is in K union, we return “never” type as its key, otherwise the key itself.
That way, we filter out the properties and got the required object type.