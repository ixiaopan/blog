---
title: "JavaScript with Promise Revisited"
date: "2023-03-04"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---


关于 `promise` 介绍的太多了，今天就推荐一本我收藏已久的小册子，[Javascript with promise](https://book.douban.com/subject/26261877/)

<!--more-->

## Run to completion and the Event loop

This chapter explains how async tasks/callbacks work.

As we all know, javascript runs on a single thread, which means you can only do one thing at one time. For the async tasks or callbacks, they are not part of the JavaScript. In other words, they run on a separate thread. 

[run to completion]

Once the async tasks are done, they need to communicate with JavaScript. But JavaScript is single-threaded, you cannot interrupt the current executing code. 

[event loop] 

One way is to put the callback into a queue. When JavaScript is available, it will look up the queue and execute each callback in order. Unfortunately, we do know how long the callback will have to wait after putting into the queue. 


## Promise


> A promise is an object that serves as a placeholder for the value of an operation.


A promise has three states

- pending: the init state

- fulfilled: the operation has completed

- rejected: the operation could not be completed

When a promise is no longer pending, it is said to be settled. In other words, that promise has reached its final state and it can never be changed.


## Common Practices

### The Async Ripple Effect

> As a general rule, any function that uses a promise should also return a promise

```js
function fetchList() {
  listAPI().then((data) => console.log(data))
}
// trigger error: fetchList() return undefined
fetchList().then(() => {})
```

### Conditional Logic

Avoid this style of conditional async execution

```js
function showMenu() {
  if (!user.authenticated) { 
    user.login().then(showMenu)
    return
  }
  
  // other things...
}
```

The problem is that `showMenu()` may run asynchronously or synchronously. The proper pattern is shown below,


```js
function showMenu() { 
  const p = (!user.authenticated) ? user.login() : Promise.resolve()
  return p.then(function () { 
    // other things...
  })
}
```

### Parallel execution

```js
const userP = ['tom', 'alice', 'bob'].map((name) => ajax(name))
const p = Promise.all(userP)
```

However, `p` will be rejected if one of the `userP` fails.


If we want to get all the settled promises regardless of whether they succeeded or failed, we can use `Promise.allSettled()`

```js
Promise.allSettled = (pList) => {
  const allP = pList.map((p) => {
    return p.then(
      (value) => {
        return { value, state: 'fulfilled' }
      }, 
      (reason) => {
        return { reason, state: 'rejected' }
      }
    )
  })

  return Promise.all(allP)
}
```


### Sequential execution

#### Using a loop

```js
function sequence(arr, asyncFunc) {
  return arr.reduce((p, val) => {
    return p.then(() => asyncFunc(val))
  }, Promise.resolve())
}
```

#### Using recursion

```js
function sequence(arr, asyncFunc) {
  const step = (idx) => {
    if (idx == arr.length) return Promise.resolve()
    
    return Promise.resolve(asyncFunc(val)).then(() => step(idx + 1))
  }
  
  step(0)
}
```

There are a little differences between the two methods.

- Using a loop builds an entire promise chain when the `sequence()` finishes, however, the recursive method builds the chain step by step.

- the recursive method doesn't need a predefined number of steps based on the elements in an array. This pattern is widely used in Nodejs.

```js
function readFile(file) {
  const reader = file.getReader()
  
  const chunks = []
  
  pump()

  const pump = () => {
    return reader.read().then((ret) => {
      if (ret.done) {
        return chunks
      }
      
      chunks.push(ret.value)
      
      return pump()
    })
  }
}
```


### Race

```js
Promise.race([p1, p2])
```

A common case is the competition between timer and request.

```js
function getData() {
  const fetchInfo = ajax()

  const getFromCache = new Promise((resolve) => {
    const cached = ''
    setTimeout(() => {
      resolve(cached)
    }, 1000)
  })
  return Promise.race([fetchInfo, getFromCache])
}

function getData() {
  const fetchInfo = ajax()

  const timeout = new Promise((resolve, reject) => {
    setTimeout(() => {
      reject()
    }, 1000)
  })

  return Promise.race([fetchInfo, timeout])
}
```

## Error Handling


### Rejecting a promise

- explicit rejection

- throw an error(explicitly/implicitly) in any of the callbacks the promise invokes (i.e., any callback passed to the Promise constructor, then, or catch.)


```js
new Promise((resolve, reject) => {
  reject(new Error('Explicit rejection'))
})

new Promise((resolve, reject) => {
  console.log(a) // implicitly throw an error
})

new Promise((resolve, reject) => {
  throw new Error('explicit throw error')// explicitly throw an error
})
```

> Any error that occurs in a function that returns a promise should be used to reject the promise instead of being thrown back to the caller. 
>
> This approach allows the caller to deal with any problems that arise by attaching a catch handler to the returned promise instead of surrounding the call in a try/catch block


就是说，要在 promise 内抛出错误，这样可以在 `catch` 中捕获，避免在调用 `promise` 的地方，使用 `try/catch`


Avoid this 

```js
function badFunc(url) { 
  var image; 
  image.src = url;  // Error: image is undefined 
  return new Promise(function (resolve, reject) { 
    image.onload = resolve; 
    image.onerror = reject;
  })
}
function a() {
  try {
    badFunc()
  } catch (e) {
    console.log(e) // TypeError: Cannot set properties of undefined (setting 'src')
  }
}
```

Recommend this

```js
function badFunc(url) { 
  return new Promise(function (resolve, reject) { 
    var image; 
    image.src = url;  // Error: image is undefined 
    image.onload = resolve; 
    image.onerror = reject;
  })
}
function a() {
  // TypeError: Cannot set properties of undefined (setting 'src')
  badFunc().catch((e) => console.log(e)) 
}
```


### Passing Errors

When we have a chain of promises, it's a common practice to add a `catch` at the end of the chain.

> Because one promise is rejected, all subsequent promises in this chain area rejected in a domino effect until an onRejected handler is found

但是使用 `catch` 有一点需要注意

> The catch function returns a new promise similar to then, but the promise that catch returns is only rejected if the callback throws an error.

换句话说，如果希望继续在 `promise chain` 中传递错误，你必须明确在 `catch` 中抛出错误才行，

```js
function catchMe() {
  queryUser()
  .then(() => queryInfo())
  .catch((err) => {
    throw err
  })
}
```

## Combine generator

主要是结合 `generator` 可以实现 `async`

```js
async function loadImage() {
  const imageData = await fetchImage()
  console.log(imageData)
}
```