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

答案：输出为 1
补充说明：在浏览器环境，这段代码输出为 1，但是在 nodejs 环境，则输出为 undefined ，在nodejs中，顶层变量不会自动挂载到全局对象global上。

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

参考答案：代码片段 1 没有 width 的过渡效果，代码 2 有 width 的过渡效果。
更优答案：代码片段 1 没有 width 的过渡效果，代码 2 有 width 的过渡效果。原因是：浏览器默认第一次渲染是不会有过渡效果的，其 render 是在第一次事件循环之后触发的，当第一次 render 执行完成之后，再对 #box 块进行 width 的修改，才会产生过渡效果。（promise 的 .then 部分是 micro task，在第一次事件循环中触发执行，所以没有产生过渡效果，而 setTimeout 中的回调属于 macro task，会在第二次事件循环中执行，并且是在第一次 render 之后执行，所以会产生过渡效果）