# Vue Series

## 相比于 Vue2.0，Vue3.0 有那些变化？

我一般会从三个方面去讲：底层实现、开发体验、性能与扩展性。

**首先是 底层实现。**
Vue3 最大的变化就是响应式系统从 Object.defineProperty 换成了 Proxy。
这样可以直接监听对象的新增、删除、数组索引这些以前做不到的情况，性能也更好，代码量更少，整体更灵活。

**第二个是 开发体验。**
Vue3 引入了 Composition API，比如 setup、ref、reactive 这些。
它主要解决了 Vue2 里 mixin 逻辑分散、复用困难的问题，让我们可以更清晰地组织代码逻辑，也更好地配合 TypeScript 使用。

**然后是 性能和打包优化。**
Vue3 整个源码是基于 ES Module 写的，天然支持 Tree-shaking，打包时只会包含用到的部分，体积更小。
另外，它的虚拟 DOM 也做了重写，比如静态节点提升、diff 算法优化，渲染效率更高。

**再有一些新的特性也挺实用的，比如：**
- Fragment 支持多根节点组件，不用再多包一层 div；
- Teleport 可以把组件内容渲染到 DOM 任意位置，比如做弹窗；
- Suspense 用来更优雅地处理异步组件加载；
- 还有 自定义渲染器 API，可以让 Vue 渲染到非 DOM 环境，比如 Canvas 或终端。

总体来说，Vue3 不只是语法更新，它是一次架构级重写，在性能、类型支持和灵活性上都比 Vue2 提升非常大。

## 为什么 Vue 3 放弃 Object.defineProperty 改用 Proxy？

1. Object.defineProperty 只能监听已有属性，新增或删除属性时需要手动处理；
2. 对数组下标变化无法劫持；
3. 深层对象需要递归遍历所有属性，初始化成本高；
4. Proxy 可以懒代理（访问到子对象时再代理），性能更优；
5. Proxy 可以监听更多类型的操作，如删除属性、获取键名等。

**简答模板：**
因为 Proxy 功能更全面、性能更好，且能天然支持数组和深层对象，所以 Vue 3 用它替代了 Object.defineProperty。

## vue2默认通过defineProperty无法监听某些数组操作，这些操作有哪些，然后vue又是怎么解决的

数组操作包括：
- 下标赋值
- pop
- push
- shift
- unshift
- splice

默认 vue2 通过 defineProperty 是无法监听的，vue2 中的解法是：
- 全局Array类的原型拷贝一份
- 对这个备份原型的方法一一进行重写，里面会去触发响应式更新，并将一些新加的对象数据进行响应式处理，同时执行原始原型中的方法来确保数组被正常操作
- 备份原型会被挂载到原始对象上面，替换原始原型
- 这样，当用户调用数组方法时，会先触发我们重写的方法，里面会去触发响应式更新，然后再执行原始原型中的方法

## vue2 中 defineProperty 无法监听索引的新增和删除，最终 vue2 是如何处理的？

vue2 补充了两个方法来手动触发响应式更新：
- Vue.set(target, index, value)
- Vue.delete(target, index)