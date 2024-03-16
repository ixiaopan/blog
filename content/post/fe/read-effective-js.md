---
title: "《Effective JavaScript》"
date: "2024-03-14"
description: ""
tags: ["Reading"]
categories : [
  "Frontend",
]
katex: true
---

## floating-point numbers

Number 类型 follow IEE 754 的规范，即双精度浮点数，

- 浮点数运算会造成误差，比如 `0.1 + 0.2 // 0.30000000000000004`
- 能表示的安全整数的范围大小在 `[-2^53 + 1, 2^53 - 1]` 之间，在此范围内的整数，精度不会丢失

在某些情况下，比如金额计算，可以考虑转成金额的最小面值计算，避免精度丢失

```js
// 元
(0.1 + 0.2) + 0.3 // 0.6000000000000001
// 分
10 + 20 + 30 // 60
```

## isNaN

`null/undefined` 参与算术运算，不同的隐式转换

- `null` 转为 `0`
- `undefined` 则转为 `NaN`

```ts
let a = null
a + 2 // 2

var b // undefined
b + 3 // NaN
```

现在要判断一个值是不是真的 `NaN`，如果使用 `isNaN()` 会发现，我们无法判断

```ts
isNaN(NaN) // true

isNaN('foo') // true
isNaN(undefined) // true
isNaN({}) // true
```

幸运的是，`NaN` 非常特殊，它不等于自身，所以我们可以利用这个特性解决上面的问题

```ts
let a = NaN
a !== a // true

function isRealNaN(x) {
  return x !== x
}
```


## toString/valueOf

对象也会发生隐式转换，比如

```js
'hi' + Math // 'hi[object Math]'
2 + { valueOf: () => 3 } // 5
```

- 转为 `string` 隐式调用了 `toString()`
- 转为 `number` 隐式调用 `valueOf()`

从上面的代码，也可以看出 `+` 既可以做加法运算，也可以实现字符串拼接，如果一个对象既实现了 `toString()` 也实现了 `valueOf()` ，该用哪个呢？ 答案是 js 优先选择 `valueOf()`


```ts
var obj = {
  toString: function () { return "[object MyObject]" }, 
  valueOf: function () { return 1 } 
}

"object: " + obj // object: 1
```

## call/apply/bind/constructor

`bind` 可以改变 `this`，不再赘述。bind 另个功能是可以实现 `currying`

```ts
function add(a, b) { return a + b }
const boundAdd = add.bind(null, 2)
boundAdd(3) // 5
```

当 bind 和 constructor 结合使用，会忽略传递的 `thisArg` 但是如果把 constructor 当做普通函数调用，`thisArg` 依然有效

```ts
function Point(x, y) {
  this.x = x
  this.y = y
}
Point.prototype.toString = function () {
  return `${this.x},${this.y}`;
}

// case 1  正常使用
const p = new Point(1, 2)
p.toString() // '1,2'

// case 2: bind with new 
const empty = {}
const BoundPoint = Point.bind(empty, 1)
const a = new BoundPoint(2)
console.log(a.toString()) // 1,2
console.log('empty:', empty) // {}

// case 3: bind without new 
const np = BoundPoint(2)
console.log('empty:', empty) // { x: 1, y: 2}
```

换句话说，改变 `this` 的优先级 `new Constructor() > bind > call/apply`

## Array.isArray

`x instanceof Array` 的问题在于，如果存在多个 Array constructor 的环境，比如 iframes，frame A 的数组在 frame B 下执行这段代码就会出错，因为 frameA/B 对应的是不同的 Array constructor，所以 frameA 的数组不会继承 frameB 的 `Array.prototype`。为了解决这个问题，ES5 引入了 `Array.isArray()` 


## concurrent operation

```ts
function downloadAllAsync(urls, onsuccess, onerror) { 
  var result = [], length = urls.length;
  
  if (length === 0) {
    setTimeout(onsuccess.bind(null , result), 0); return
  }

  urls.forEach(function (url) { 
    downloadAsync(
      url, 
      (text) => {
        if (result) {
          // race condition
          result.push(text)

          if (result.length === urls.length) { 
            onsuccess(result)
          }
        }
      }, 
      (error) => {
        if (result) { 
          result = null
          onerror(error)
        }
      }
    )
  });
}
```

如上的代码会发生 `data racing` 即 result 数据顺序和 urls 顺序不一致，解决方法是，加上索引

```ts
urls.forEach(function (url, i) { 
  // ...
  result[i] = text
  if (result.length === urls.length) { 
    onsuccess(result)
  }
})
```

但依然有问题，问题在于 `arr[i]` 会自动修改 `length = i + 1`，如果 urls 最后一个 url 第一个完成下载，那这段代码会立即触发 `onsuccess(result)`

解决方法是，通过 counter 追踪当前 pending request 的数量

```ts
let pending = urls.length // 新增

urls.forEach(function (url, i) { 
  // ...
  result[i] = text
  pending-- // 新增
  if (pending === 0) { // 新增
    onsuccess(result)
  }
})
```

## Refer
- [Double-precison floating-point format](https://zhuanlan.zhihu.com/p/351127362)
- [Function.prototype.bind - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
