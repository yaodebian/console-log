# Code Series

## 实现一个简单的柯理化函数

```js
function curry(fn) {
  const context = this
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(context, args)
    }
    return function (...restArgs) {
      return curried.apply(context, args.concat(restArgs))
    }
  }
}
```