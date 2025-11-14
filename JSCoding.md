# Code Series

## 现在给出一个数组 [1，2，3，4]，想要生成一个新的数组，里面的数值大小是原来的两倍。

```js
function doubleArray(arr) {
  const resArr = []
  for (let i = 0; i < arr.length; i++) {
    resArr.push(arr[i] * 2)
  }
  return resArr
}

console.log(doubleArray([1, 2, 3, 4])) // [2, 4, 6, 8]
```

更优解：（使用高阶函数 Array.prototype.map）

```js
function doubleArray(arr) {
  return arr.map(item => item * 2)
}

console.log(doubleArray([1, 2, 3, 4])) // [2, 4, 6, 8]
```

## 现在给出一个数组 [1, 2, 3, 2, 4, 6, 3, 6, 1]，想要生成一个新数组，这个数组是原来数组去重后所得。

方法一：
```js
function deduplicationArr (arr) {
  const resArr = []
  for (let i = 0; i < arr.length; i++) {
    if (i === arr.indexOf(arr[i])) {
      resArr.push(arr[i])
    }
  }
  return resArr
}

console.log(deduplicationArr([1, 2, 3, 2, 4, 6, 3, 6, 1]))
```

方法二：（借助对象）
```js
function deduplicationArr (arr) {
  const obj = {}
  for (let i = 0; i < arr.length; i++) {
    obj[arr[i]] = arr[i]
  }
  return Object.values(obj)
}

console.log(deduplicationArr([1, 2, 3, 2, 4, 6, 3, 6, 1])) // [ 1, 2, 3, 4, 6 ]
```

方法三：（使用高阶函数 Array.prototype.reduce）

```js
function deduplicationArr (arr) {
  return arr.reduce((pre, cur) => {
    if (pre.includes(cur)) {
      return pre
    }
    return [...pre, cur]
  }, [])
}

console.log(deduplicationArr([1, 2, 3, 2, 4, 6, 3, 6, 1])) // [ 1, 2, 3, 4, 6 ]
```

方法四：(使用高阶函数 Array.prototype.filter)

```js
function deduplicationArr (arr) {
  return arr.filter((item, index) => arr.indexOf(item) === index)
}

console.log(deduplicationArr([1, 2, 3, 2, 4, 6, 3, 6, 1])) // [ 1, 2, 3, 4, 6 ]
```

方法五：（使用 Set 数据结构）

```js
function deduplicationArr (arr) {
  return [...new Set(arr)]
}

console.log(deduplicationArr([1, 2, 3, 2, 4, 6, 3, 6, 1])) // [ 1, 2, 3, 4, 6 ]
```

针对对象数组去重，你会怎么实现呢？

```js
const list = [
  {
    name: '海贼王',
    score: 10,
  },
  {
    name: '火影',
    score: 10,
  },
  {
    name: '黑色五叶草',
    score: 10,
  },
  {
    name: '死神',
    score: 10,
  },
  {
    name: '死神',
    score: 10,
  },
  {
    name: '火影',
    score: 10,
  },
  {
    name: '黑色五叶草',
    score: 10,
  },
]

// method 01
const filterKeys = list.map(item => item.name)
const finalList = list.filter((item, index) => filterKeys.indexOf(item.name) === index)
console.log(finalList)

// method 02
const temp = {}
const finalList02 = []
for (const item of list) {
  if (temp[item.name]) {
    continue
  }
  temp[item.name] = item.name
  finalList02.push(item)
}
console.log(finalList02)

// method 03
const temp02 = {}
const finalList03 = list.reduce((accumulator, currentValue)  => {
  if (!temp02[currentValue.name]) {
    temp02[currentValue.name] = currentValue.name
    accumulator.push(currentValue)
  }
  return accumulator
}, [])

console.log(finalList03)
```

## 给出一个数字数组 [0, 1, 2, 3, 4]，请计算数组中所有项的和？

方法一：
```js
function sum (arr) {
  let res = 0
  for (let i = 0; i < arr.length; i++) {
    res += arr[i]
  }
  return res
}

console.log(sum([1, 2, 3, 4])) // 10
```

方法二：（使用高阶函数 Array.prototype.reduce）

```js
// 不设置初始值时，reduce 方法会从索引 1 开始执行回调函数，将索引 0 的值作为初始值
function sum (arr) {
  return arr.reduce((accumulator, currentValue, currentIndex, array) => {
    return accumulator + currentValue
  })
}

console.log(sum([1, 2, 3, 4])) // 10
```

```js
// 设置初始值时，reduce 方法会从索引 0 开始执行回调函数，将初始值作为累加器的初始值
function sum (arr) {
  return arr.reduce((accumulator, currentValue, currentIndex, array) => {
    return accumulator + currentValue
  }, 0)
}

console.log(sum([1, 2, 3, 4])) // 10
```

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

## 实现一个支持占位符的柯里化函数

演示案例：
```js
function join(a, b, c) {
  return `${a}_${b}_${c}`
}
const curriedJoin = curry(join)
const _ = curry.placeholder
curriedJoin(1, 2, 3) // '1_2_3'
curriedJoin(_, 2)(1, 3) // '1_2_3'
curriedJoin(_, _, _)(1)(_, 3)(2) // '1_2_3'
```

参考解法：
```js
function curry(fn) {
  const context = this
  return function curried(...args) {
    const isComplete = args.length >= fn.length && !args.slice(0, fn.length).includes(curry.placeholder)
    if (isComplete) {
      return fn.apply(context, args)
    }
    return function (...restArgs) {
      const argList = args.map(item => {
        return item === curry.placeholder && restArgs.length 
          ? restArgs.shift()
          : item
      })
      return curried.apply(context, argList.concat(restArgs))
    }
  }
}
```
