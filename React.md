# React Series

## useEffect 和 useLayoutEffect 的区别能简单描述一下吗？

useEffect 是异步执行的，不阻塞渲染，但容易造成闪动；useLayoutEffect 是同步执行的，会阻塞渲染，造成卡顿，所以一般不用 useLayoutEffect。