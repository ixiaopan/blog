---
title: "Promise/Async/Generator"
date: "2024-03-15"
description: ""
categories : [
  "Frontend",
]
---

## Simple callback

difficult error handling because `try...catch` don't work

```ts
getJSON('xx.json', (err, data) => {
  if (err) console.log('err')
  // do something
})
```

sequences of steps, leading to pyramid of doom

```ts
getJSON('xx.json', (err, data) => {
  if (err) console.log('err')
  
  invite(data[0].name, (err, data) => {
    if (err) console.log('err')
    // do something
  })
})
```

waiting until all the parallel tasks are done

```ts
let person, item,
getPerson('xx', (err, data) => {
  if (err) console.log('err')
  person = data
  checkReady()
})
getItem('xx', (err, data) => {
  if (err) console.log('err')
  item = data
  checkReady()
})
function checkReady() {
  if (!person && !item) {
    console.log('ready go')
  }
}
```


## Promise

### executor

```ts
console.log('1')
let promise = new Promise(function(resolve, reject) {
  console.log('2')
  
  setTimeout(() => {
    resolve("done")
    resolve('ok') // ignored
    reject("my error") // ignored
  }, 1000)
})
promise.then((val) => {
  console.log('val: ', val)
})
// 1, 2, val: done
```

- `executor()` 会同步执行
- `executor()` 接收2个参数：`resolve, reject`
  - 任务成功调用 `resolve(val)`
  - 失败则调用 `reject(err)`
- 一旦调用 `resolve` 或者 `reject`，再次调用都会被忽略，换句话说，此时 `promise is settled`，状态不会再发生变化

简单实现如下

```ts
const PEDNING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  constructor(executor) {
    this.state = PEDNING
    
    this._resolvedQueue = []
    this._rejectedQueue = []

    const _resolve = (v) => {
      if (this.state !== PEDNING) return

      this.state = FULFILLED
      this._resolvedQueue.forEach((f) => f(v))
    }

    const _reject = (err) => {
      if (this.state !== PEDNING) return
      
      this.state = REJECTED
      this._rejectedQueue.forEach((f) => f(err))
    }

    executor(_resolve, _reject)
  }

  then(onFulfilled, onRejected) {
    this._resolvedQueue.push(onFulfilled)
    this._rejectedQueue.push(onRejected)
  }
}
```

但是，`executor` 不一定是异步操作，比如任务已经完成，直接从缓存获取

```ts
let promise = new Promise(function(resolve, reject) {
  resolve("done")
})
```

目前 `MyPromise` 无法支持，还需要进一步改进，一种方式是让 `resolve` 内部走一个异步，比如 setTimout

```ts
const _resolve = (v) => {
  const step = () => {
    if (this.state !== PEDNING) return
    this.state = FULFILLED
    this._resolvedQueue.forEach((f) => f(v))
  }

  setTimeout(step)
}
```

### chaining

目前 `MyPromise.then()` 还过于简陋，标准的 `then` 链式调用还未实现

- then 返回新的 promiseA
- onFulfilled 返回值
  - 返回另一个 promiseB，那么 promiseA 以 promiseB 的状态为准
  - 返回非 promise 值，会带入下一个 `then(onFulfilled)` 中

```ts
then(onFulfilled, onRejected) {
  return new Promise((resolve, reject) => {
    const _fulfilled = (val) => {
      const ret = onFulfilled(val)
      ret instanceof MyPromise ? ret.then(resolve, reject) : resolve(ret)
    }
    this._resolvedQueue.push(_fulfilled)
  })
}
```

测试一下

```ts
new MyPromise(function(resolve, reject) {
  setTimeout(() => { resolve("done") }, 1000)
})
.then((val) => {
  console.log('val: ', val) // val: done
  return new MyPromise((resolve) => resolve(val + 'seconds'))
})
.then((val) => console.log('second val: ', val)) // doneseconds
```

有时候，也不需要返回真正的 promise 对象，比如一个拥有 `then` 的对象

```ts
then(onFulfilled, onRejected) {
  return new Promise((resolve, reject) => {
    const _fulfilled = (val) => {
      // ret instanceof MyPromise ? ret.then(resolve, reject) : resolve(ret)
      // duck type
      ret.then ? ret.then(resolve, reject) : resolve(ret)
    }
    this._rejecteddQueue.push(onRejected)
  })
}

// 测试一下
class Thenable {
  constructor(name) {
    this.name = name;
  }
  then(resolve, reject) {
    setTimeout(() => resolve('thenable: ' + this.name), 1000)
  }
}

promise
.then((val) => {
  console.log('val: ', val)
  // return new MyPromise((resolve) => resolve(val + 'seconds'))
  return new Thenable(val)
})
.then((val) => {
  console.log('second val: ', val) // second val:  thenable: done
})
```

### error handling

```ts
// code 1
promise.then(fn, error).then(() => console.log('fulfilled'))

// code 2 in a promise chain, the last catch will catch all error
promise.then(fn).then(fn).catch(error)

// code 3 passing errors
promise().catch((err) => { throw err })

// code 4 implicit try...catch in executor
new Promise((resolve) => {
  throw new Error('err')
}).catch(console.log)

// code 5
new Promise(function(resolve, reject) {
  setTimeout(() => {
    throw new Error("Whoops!")
  }, 1000);
}).catch(console.log) // not catched

// code 6
window.addEventListener('unhandledrejection', function(event) {})
new Promise(function() {
  throw new Error("Whoops!")
})
```

```ts
then(onFulfilled, onRejected) {
  return new Promise((resolve, reject) => {
    const _rejected = (val) => {
      try {
        if (!onRejected) {
          reject()
        } else {
          const ret = onRejected(val)
          // 注意这里的 resolve()
          ret && ret.then ? ret.then(resolve, reject) : resolve(ret)
        }
      } catch (e) {
        reject(e)
      }
    }
    this._rejectedQueue.push(_rejected)
  })
}
catch(cb) {
  return this.then(undefined, cb)
}
```

### finally

- `finally()` 没有入参，会自动把上个 promise 传递下去
- 忽略 handler 内部的返回值
- handler 内部抛出错误，被最近的 error handler 捕获


```ts
// pass result
new Promise((resolve, reject) => {
  setTimeout(() => resolve('done'), 2000);
})
.finally(() => Promise.resolve('ds'))
.then(result => console.log(result)) // done

// pass error
new Promise((resolve, reject) => {
  throw new Error("error")
})
.finally(() => console.log("finally"))
.catch(err => console.log(err))

// throws error
new Promise((resolve, reject) => {
  throw new Error("error")
})
.finally(() => { throw new Error("ds") })
.catch(err => console.log(err)) // ds
```

实现也比较简单

```ts
finally(cb) {
  return this.then(
    (val) => {
      return MyPromise.resolve(cb()).then(() => val)
    },
    (e) => {
      return MyPromise.resolve(cb()).then(() => { throw e })
    }
  )
}
```

### API

```ts
class MyPromise {
  static resolve(val) {
    if (v instanceof MyPromise) return v
    return new MyPromise((resolve) => resolve(v))
  }
  static reject(err) {
    return new MyPromise((_, rej) => rej(err))
  }
  static all(asyncList) {
    let result = []
    let counter = asyncList.length
    
    return new MyPromise((resolve, reject) => {
      asyncList.forEach((p, i) => {
        p.then(
          (v) => {
            result[i] = v
            counter--

            if (counter === 0) {{
              resolve(result)
            }}
          }, 
          (e) => {
            reject(e)
          }
        )
      })
    })
  }
  static race() {
    return new MyPromise((resolve, reject) => {
      asyncList.forEach((p, i) => {
        p.then(
          (v) => {
            resolve(v)
          }, 
          (e) => {
            reject(e)
          }
        )
      })
    })
  }
  static allSettled() {}
  static any() {}
}
```

## Iterator

- iterable 决定是否可以 iteration
- iterator 定义 how iteration

### iterator

> The iterator protocol defines a standard way to produce a sequence of values (either finite or infinite), and potentially a return value when all values have been generated.

`iterator` 是一个通过 `next()` 实现 [Iterator protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#the_iterator_protocol) 的对象，`next()` 会返回一个具有 `{ value: any: done: boolean}` 的对象。`iterator` 需要手动调用 `next()` 会进行遍历，直到返回 `{ done: true }` 结束


```ts
function rangeIterator(min, max, step = 1) {
  let start = min
  
  return {
    next() {
      if (start < max) {
        const ret = { value: start, done: false }
        start += step
        return ret
      }
      return { value: max, done: true }
    }
  }
}
```

### iterable

> The iterable protocol allows JavaScript objects to define or customize their iteration behavior, such as what values are looped over in a for...of construct.

一个实现了 `@@iterator` 方法的对象，即这个对象拥有 `Symbol.iterator` 属性，返回遵循 `iterator protocol` 的对象

- 只能遍历一次（比如 `generator`），每次调用 `@@iterator` 都返回 `this`
- 也可以被遍历多次，每次调用 `@@iterator` 都会返回一个新的 `iterator`
- 内置的 `iterable object`: `String, Array, Map, Set...`
- 内置的 `iterable API`: `Promise.all(), Array.from(), Map(), Set()...`
- `@@iterator` 可以是普通函数、generator

```ts
// case1 generator itselft is iterable, and it can iterate only once
function* step() {
  yield 1;
  yield 2;
}
const iterA = step()
console.log('a:', iterA.next()) // { value: 1, done: false }
const iterB = step()
console.log('b:', iterB.next()) // { value: 1, done: false }
console.log('b:', iterB.next()) // { value: 2, done: false }
console.log('a:', iterA.next()) // { value: 2, done: false }


// case2 retuns a new generator, which can iterate many time
const obj = {
  *[Symbol.iterator]() {
    yield 1
    yield 2
    yield 3
  }
}

const iter1 = obj[Symbol.iterator]()
console.log('iter1: ', iter1.next()) // { value: 1, done: false }
console.log('iter1: ', iter1.next()) // { value: 2, done: false }

const iter2 = obj[Symbol.iterator]()
console.log('iter2: ', iter2.next()) // { value: 1, done: false }

// case3 普通函数 retuns a new iterator, which can iterate many time
const obj = {
  [Symbol.iterator]() {
    let start = 1
    return {
      next() {
        if (start < 3) {
          return { value: start++, done: false }
        }
  
        return { value: undefined, done: true }
      }
    }
  }
}
```


哪些情况是以 iterables 作为参数？

```ts
for (const value of ["a", "b", "c"]) {
  console.log(value);
}

[..."abc"]

[a, b, c] = new Set(["a", "b", "c"])
```

## Generator
> Regular functions return only one, single value (or nothing).
>
> Generators can return (“yield”) multiple values, one after another, on-demand. They work great with iterables, allowing to create data streams with ease.

### yield

- generator is a special type of iterator，and it's iterable.
- `yield` returns the result to the outside and passes the value to the generator
- `next()` supplies the value to the waiitng `yield`, so if there is no `yield` waiting, this is no need to supply the value to. In other words, the first call `next()` has no argument.


```ts
function* sum(a) {
  const b = yield 2 + a
  const c = yield b + 5
  return c
}
const iterSum = sum(1) // pass parameter like other normal functions do
console.log(iterSum.next()) // 3, the first call `next()` has no argument.
console.log(iterSum.next(4)) // 9
console.log(iterSum.next(12)) // 12
```

### throw

```ts
function* sum(a) {
  try {
    const b = yield 2 + a
    const c = yield b + 5
    return c
  } catch (e) {
    console.log('error:', e.message) // error: custom error
  }
}
const iterSum = sum(1)
console.log(iterSum.next()) // 3
iterSum.throw(new Error('custom error'))
```

如果不在 `generator` 内捕获的话，error 会抛在调用 generator 的地方

```ts
function* sum(a) {
  const b = yield 2 + a
  const c = yield b + 5
  return c
}
const iterSum = sum(1)
console.log(iterSum.next()) // 3
try {
  iterSum.throw(new Error('custom error'))
} catch (e) {
  console.log('error:', e.message) // error: custom error
}
```

### under the hood

#### state machine

![](/blog/post/images/generator-state.png "Source From: Secrets of the JavaScript Ninja-2nd")

<i>Source From: Secrets of the JavaScript Ninja-2nd</i>

- `suspended start`: 调用 `generator()` 进入起始状态
- `executing`: 调用 `next()` 进入此状态，即初次运行或者从上次 `suspended` 的地方继续
- `suspended yield`: 遇到 `yield`
- `completed`: 遇到 `return` 或者函数结束


#### execution context stack

以这段代码为例（以下代码和图来源于 《Secrets of the JavaScript Ninja-2nd》）

```ts
function* NinjaGenerator(action) { 
  yield "Hattori " + action
  return "Yoshi " + action
}

const ninjaIterator = NinjaGenerator("skulk")
const result1 = ninjaIterator.next()
const result2 = ninjaIterator.next()
```

Step 1: `NinjaGenerator("skulk")`

![](/blog/post/images/generator-step-1.png)
![](/blog/post/images/generator-step-2.png)

Step 2: `const result1 = ninjaIterator.next()`

![](/blog/post/images/generator-step-3.png)
![](/blog/post/images/generator-step-4.png)

Step 3: `const result2 = ninjaIterator.next()`

同 `Step 2`


## Async/Await

利用 `generator` 简单实现 `async/await`

```ts
function run(gen) {
  const iter = gen()
  
  const step = (val) => {
    const res = iter.next(val)

    if (res.done) return
    
    res.value.then((v) => { step(v) }).catch(err => iter.throw(err))
  }

  step()
}

run(function *() {
  try {
    const a = yield Promise.resolve(1)
    console.log('a', a)
    
    const b = yield Promise.resolve(3)
    console.log('b', b)
  } catch (e) {
    console.log('opps')
  }
})
```

## See also
- [promise, async - javascript.info](https://javascript.info/async)
- [Iterators and Generators - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)
- [generators - javascript.info](https://javascript.info/generators-iterators)
- [9k字 | Promise/async/Generator实现原理解析 - 掘金](https://juejin.cn/post/6844904096525189128)
