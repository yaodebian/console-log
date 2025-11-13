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


