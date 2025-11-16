# Javascript Series

## 作用域相关：请问以下代码的打印结果是什么？

```js
var num = 1

function A () {
  console.log(this.num)
}

function B () {
  var num = 2

  A()
}

B()
```

- 答案：输出为 1
- 补充说明：在浏览器环境，这段代码输出为 1，但是在 nodejs 环境，则输出为 undefined ，在nodejs中，顶层变量不会自动挂载到全局对象global上。

## event loop 相关：说一下以下两段代码实现效果的区别（考察宏任务 macro task 与微任务 micro task）

代码片段01:
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>test</title>
  <style>
    #box {
      width: 100px;
      height: 100px;
      background-color: yellow;
      transition: width 2s;
    }
  </style>
</head>

<body>
  <div id="box"></div>
  <script>
    let box = document.querySelector('#box')
    new Promise((resolve, reject) => {
      resolve()
    }).then(() => {
      box.style.width = '200px'
    })
  </script>
</body>

</html>
```

代码片段02:
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>test</title>
  <style>
    #box {
      width: 100px;
      height: 100px;
      background-color: yellow;
      transition: width 2s;
    }
  </style>
</head>

<body>
  <div id="box"></div>
  <script>
    let box = document.querySelector('#box')
    setTimeout(() => {
      box.style.width = '200px'
    })
  </script>
</body>

</html>
```

- 参考答案：代码片段 1 没有 width 的过渡效果，代码 2 有 width 的过渡效果。
- 更优答案：代码片段 1 没有 width 的过渡效果，代码 2 有 width 的过渡效果。原因是：浏览器默认第一次渲染是不会有过渡效果的，当第一次 render 执行完成之后，再对 #box 块进行 width 的修改，才会产生过渡效果。（渲染是在事件循环之后触发的，promise 的 .then 部分是 micro task，在第一次事件循环中触发执行，所以没有产生过渡效果，而 setTimeout 中的回调属于 macro task，会在第二次事件循环中执行，并且是在第一次 render 之后执行，所以会产生过渡效果）

## Javascript 的 event loop（事件循环）了解吗，请简单描述一下

参考答案：
首先代码操作有同步操作与异步操作，异步操作分为宏任务与微任务，我们可以把主线程的同步操作看成是刚从任务队列取出的宏任务，假如它的执行过程中产生了宏任务与微任务，它会把宏任务添加进任务队列，微任务添加进微任务队列，当代码执行完之后，就会取出所有的微任务队列执行，所有可执行的微任务执行完毕之后，开始渲染。渲染完之后，接着取出任务队列中的第一个宏任务，重复上面的操作。

## js中的基础数据类型有哪些

参考答案：
js中的基础数据类型有 number、string、boolean、null、undefined、symbol、bigint

## Symbol的用途是什么

参考答案：Symbol 是 ES6 引入的一种新的 原始数据类型，用于创建 唯一且不可变的标识符。它的出现主要是为了解决对象属性名冲突的问题，并支持更底层的语言能力（自定义对象的行为，如通过Symbol.iterator 让一个普通对象变得“可迭代”）。

```js
const myObj = {
  data: [1, 2, 3],
  [Symbol.iterator]() {
    let i = 0;
    const arr = this.data;
    return {
      next() {
        return i < arr.length
          ? { value: arr[i++], done: false }
          : { done: true };
      }
    };
  }
};

for (const x of myObj) {
  console.log(x);
}
// 输出：1, 2, 3
```

## 什么是长任务（long task），一般你又是怎么解决的？

某个任务持续时间超过 50ms，就被称为长任务（long task）。长任务可以是js任务、样式计算、布局、绘制、合成等。

当遇到长任务时，一般我们可以从几个方向去解决：
- 拆分大任务 → 用 requestIdleCallback 分批执行后台任务或者一些低优先级任务，用 requestAnimationFrame 分批执行渲染任务；
- 多线程执行 → 可以放入 Web Worker 去并发执行一些密集型任务；
- 虚拟滚动（Virtual Scrolling）→ 当列表或网格非常长时，只渲染可见区域的内容，而不是全部渲染，从而提高性能。
- 分片渲染（Chunk Rendering）→ 如果数据量不是特别大（比如几千条），可以不用虚拟滚动，而采用 任务分片（chunk）。也就是每一帧只渲染一部分，利用空闲时间分批插入 DOM。
- 代码优化 → 减少不必要的循环、DOM 操作、样式重排等。

可以通过用 PerformanceObserver 来监听 longtask。

## 什么是函数柯里化

通俗来讲就是：只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。我们把这样的一种编程思想称之为函数柯里化。

函数柯里化可以用来干嘛呢？比较典型的场景是参数复用：先传递部分参数生成一个新函数，这个新函数可以多处复用，这种常常在一些计算方面会用到。

## 说说 Object.defineProperty 与 Proxy 的区别

**覆盖面上：**
Object.defineProperty 是基于属性级别的劫持，只能监听 get 和 set，而且只能作用于已存在的属性；
Proxy 是对象级别的代理，可以拦截包括 get、set、has、deleteProperty 等在内的所有语言级操作，并且可以通过递归实现深层监听。

**触发时机：**
Object.defineProperty 的触发点在原本的对象上，而 Proxy 的触发点在 Proxy 实例上。

**中间态操作：**
Object.defineProperty 的 set 和 get 不能直接操作原有对象，需要通过一个中间状态来进行存储，否则会导致无限循环。而 Proxy 则没有这个问题，因为它的触发点在 Proxy 实例上，而不是在原有对象上。

## 为什么 Object.defineProperty 的 get/set 容易造成死循环？

Object.defineProperty 的触发点在原本的对象上，如果在set或者get的callback里面直接操作原本的对象，就会再次触发set和get。
而Proxy的触发点在Proxy实例上，在拦截到并进行原始对象的操作时，就不会再次触发了。

## Proxy 能实现“深层代理”吗？怎么实现？

- Proxy 默认只拦截第一层；
- 可以在 get 时判断返回值是否是对象；
- 若是对象，则再返回一个新的 Proxy，实现 递归代理（或懒代理）。

具体实现如下：

```js
function deepProxy(target) {
  if (typeof target !== 'object' || target === null) {
    return target
  }
  return new Proxy(target, {
    get(target, key) {
      const value = target[key]
      // do something here
      console.log(`get ${key}`)
      return deepProxy(value) // ✅ 懒递归
    },
    set(target, key, value) {
      // do something here
      console.log(`set ${key} to ${value}`)
      target[key] = value // ✅ 懒递归
      return true
    }
  })
}

const proxy = deepProxy({ a: { b: { c: 1 } } })
proxy.a.b.c       // get a → get b → get c
proxy.a.b.c = 2   // get a → get b → set c to 2
```

## CommonJS 与 ES Module 的区别

语法不同：
- CommonJS 通过 module.exports 导出模块，通过 require 引入模块；
- ES Module 通过 export 导出模块，通过 import 引入模块。

加载方式：
- CommonJS 
  - 在运行时同步加载模块
  - 它的加载是动态的，可以在块级作用域内使用
- ES Module
  - 在编译时进行静态分析，解析module graph，之后基于graph深度优先遍历的方式提前加载并执行module，然后再开始执行非import的代码。同时，它相比 CommonJs，支持异步加载（`import()`）
  - import标识符的加载是静态的，不能在块级作用域内使用，强制在模块顶部使用。

Tree Shaking 与 代码拆分（Code Splitting）支持：
- CommonJS 没有静态分析，所以不支持
- ES Module 在编译时进行静态分析，所以支持 Tree Shaking 和 Code Splitting

模块This
- CommonJS 中，每个模块都有自己的 this 指向，这个 this 指向的是module.exports对象
- ES Module 在严格模式下运行，所以 this 指向 undefined

## 闭包了解吗，请谈谈你对闭包的理解

MDN 中的定义：函数以及声明该函数的词法环境组合就是闭包。它是一种环境，在这个环境中，闭包函数可以访问其祖先作用域中的变量与方法。所以，只要一个函数它声明了，就会产生闭包。

当然，有些地方还会这样定义闭包：
- 闭包是内部函数引用外部函数的变量的集合；
- 闭包是有权访问另一个函数中变量的函数；（JS 高程）

个人更倾向于第三种定义。

闭包容易产生内存泄漏，因为闭包会引用外部函数的变量，而这些变量不会被垃圾回收机制回收。

当然闭包也有一定的应用场景，比如我们可以通过闭包来模拟私有属性，也可以实现柯里化等高阶函数。

## 给出一个函数 fn，fn.length 表示的是什么？

如果函数 fn 的形参均没有默认值，则 fn.length 表示的是 fn 形参的个数，否则表示的是从左往右数第一个带默认值的形参前面的形参个数。

```js
function test(a, b, c) {}
console.log(test.length) // 3
```

```js
function test(a, b, c = 3) {}
console.log(test.length) // 2
```

## 函数防抖与函数节流了解过吗，请谈谈你对它们的了解？

函数防抖侧重于：针对某一时间间隔，连续的事件触发，只会执行最后一次。
函数节流侧重于：针对某一固定时间范围，连续的事件触发，只执行一次。

应用场景上：
- 函数防抖常用于搜索框输入只在用户停止的输入的时候触发。
- 函数节流常用于滚动事件、resize事件、拖拽事件等场景。

## setTimeout、Promise、Async/Await 的区别

setTimeout 的回调是宏任务，用于延迟执行。
Promise 通过 then、catch 等方法来处理异步操作的结果，这里面的回调时微任务。
Async/Await 是基于 Promise 的语法糖，它可以让Promise的异步操作看起来像同步操作一样，代码管理更清晰。
