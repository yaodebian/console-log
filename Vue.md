# Vue Series

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