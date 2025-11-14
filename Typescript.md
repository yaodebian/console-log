# Typescript Series

## 两个函数f0和f1，f1是f0的入参，我如果要获取f1入参的类型，ts怎么实现

在 TypeScript 中，可以通过 Parameters 类型来获取函数的入参类型。

假设f1传递一个字符串参数，那我们在f0中通过 `Parameters<typeof f1>` 来获取类型元组 `[string]`。

```ts
// 定义 f0 和 f1
function f0(f1: (x: string) => number) {
  // 使用 f1 参数的类型
  type F1Params = Parameters<typeof f1>; // 获取 f1 的参数类型
  // F1Params 就是 [string] 类型
}
```

## 请自己实现一个 Partial<T> ，它返回一个类型，该类型表示 T 类型的所有子集。

示例：

```ts
type Foo = {
  a: string
  b: number
  c: boolean
}
// below are all valid
const a: MyPartial<Foo> = {}
const b: MyPartial<Foo> = {
  a: 'BFE.dev'
}
const c: MyPartial<Foo> = {
  b: 123
}
const d: MyPartial<Foo> = {
  b: 123,
  c: true
}
const e: MyPartial<Foo> = {
  a: 'BFE.dev',
  b: 123,
  c: true
}
```

实现：

```ts
type MyPartial<T> = {
  [P in keyof T]?: T[P]
}
```

## 请自己实现一个 Required<T> ，它返回一个类型，该类型表示 T 类型的所有属性都是必填的。

示例：

```ts
// all properties are optional
type Foo = {
  a?: string
  b?: number
  c?: boolean
}
const a: MyRequired<Foo> = {}
// Error
const b: MyRequired<Foo> = {
  a: 'BFE.dev'
}
// Error
const c: MyRequired<Foo> = {
  b: 123
}
// Error
const d: MyRequired<Foo> = {
  b: 123,
  c: true
}
// Error
const e: MyRequired<Foo> = {
  a: 'BFE.dev',
  b: 123,
  c: true
}
// valid
```

实现：
```ts
type MyRequired<T> = {
  [P in keyof T]-?: T[P]
}
```

## 请自己实现一个 Readonly<T> ，它返回一个类型，该类型表示 T 类型的所有属性都是只读的。

示例： 

```ts
type Foo = {
  a: string
}
const a:Foo = {
  a: 'BFE.dev',
}
a.a = 'bigfrontend.dev'
// OK
const b:MyReadonly<Foo> = {
  a: 'BFE.dev'
}
b.a = 'bigfrontend.dev'
// Error
```

实现：
```ts
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P]
}
```

## 请自己实现一个 Record<K, V> ，它键为 K 且值为 V 的对象类型。

示例：

```ts
type Key = 'a' | 'b' | 'c'
const a: Record<Key, string> = {
  a: 'BFE.dev',
  b: 'BFE.dev',
  c: 'BFE.dev'
}
a.a = 'bigfrontend.dev' // OK
a.b = 123 // Error
a.d = 'BFE.dev' // Error
type Foo = MyRecord<{a: string}, string> // Error
```

实现：
```ts
type ValidKeyType = string | number | symbol;

type MyRecord<K extends ValidKeyType, V> = {
  [X in K]: V
}
```

