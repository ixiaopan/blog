---
title: "Polling"
date: "2024-01-04"
description: ""
# tags: []
categories: [
  "Frontend",
]
---

业务中或多或少会遇到服务端无法立刻返回结果的情况，一般解决方案是轮询接口 `Polling`，但除此之外，还有其他数据推送方案，比如 Long Polling、SSE(Server-Sent Events) 以及 `WebSockets`。这些方案的目的都是希望可以及时/实时获取服务端数据，特别是对 real-time app 比如 chat、game，实时性是第一位。

## Short Polling

一般业务中碰到的轮询大都是 `short polling`，即每隔 `n` 秒发送一次请求，根据返回结果判断是否结束轮询，比如查询文件的上传进度。通常，我们会写如下代码

```ts
function step() {
  setTimeout(() => {
    request(url)
      .then((data) => {
        if (data.progress == 100) {
          // 正常业务处理...
        } else {
          step()
        }
      })
      .catch(() => {
        step()
      })
  }, 10 * 1000)
}
step()
```

缺点很明显，不间断的询问服务端，给服务端造成压力，也浪费 `HTTP` 和带宽，不适合业务密集型、实时度要求高的应用。

即使业务可以使用 `short polling`，上面的代码也需要进一步改进

### 超时时间

就如 `xhr` 一样，某些情况下，轮询可能也需要设置一个最大等待时间，避免内存、性能问题等。

```ts
function createPolling(timeout: number) {
  // 新增
  let startTime = Date.now()

  const step = () => {
    if (Date.now() - startTime >= timeout) return // 新增
    timer = setTimeout(() => {  })
  }
  step()
}
```

### 取消轮询

除了设置超时时间，有很多种情况，需要用户主动取消轮询，比如离开当前页面，从而避免内存泄露。

```ts
function createPolling(timeout: number) {
  let canceled = false // 新增

  const step = () => {
    if (canceled) return // 新增

    if (Date.now() - startTime >= timeout) return
    timer = setTimeout(() => {  })
  }

  // 新增
  function cancel() {
    canceled = true
    clearTimeout(timer)
  }
  return {
    step,
    cancel,
  }
}
```

### cancelToken

`step()` 存在冗余，仔细观察就能发现，`step()` 会在2个地方自动触发
  - 接口失败 `promise.catch` 
  - 未满足业务条件 `promise.then()` 中 `if not`

这2个都可以优化为 `promise.finally`，因为 `promise.finally` 不管是 `then/catch` 最后都会执行。

```ts
setTimeout(() => {
  request().then(() => {
    if (data.progress == 100) {
    }
  }).finally(step)
})
```

那问题来了，`if (data.progress == 100)` 也会走到 `finally(step)`，怎么办？这种情况，需要业务主动调用 `cancel()` 取消轮询。

```ts
setTimeout(() => {
  request().then(() => {
    if (data.progress == 100) {
      cancel() // 新增
    } 
  }).finally(step)
}
```

换句话说，业务方只需要关心**何时满足业务条件**，此时，取消轮询即可。


`step()` 冗余的问题解决了，但 `createPolling()` 本身和 `request(url).then()` 业务逻辑耦合在一起，如果有另外的轮询，我们要复制整个 `createPolling()` 但只是修改其中的 `request(url).then()` 部分。为此，我们要把 `request(url).then()` 抽象出来


```ts
function createPolling(fn, interval = 10 * 1000, timeout?: number) {
  // 新增 cancelToken
  const _run = () => fn(cancel).finally(step)

  const step = () => {
    timer = setTimeout(_run, interval)
  }

  return { step, cancel }
}

// 业务使用
const { step } = createPolling((cancel) => {
  return request(url).then((data) => {
    if (data.progress == 100) {
      cancel()
    } 
  })
})
step()
```

这段代码参考了 `axios.cancelToken` 的设计，把 `cancel` 传给业务函数 `fn`，这也是因为 `createPolling()` 虽然可以返回 `cancel` 但此时 `fn` 已经无法引用到。

## Long Polling

- 客户端发起请求
- 服务端挂起连接，直到有消息要给客户端
- 客户端收到后，立刻发起新的请求

同样，连接会有超时，或者网络错误导致连接中断，此时客户端离开发起新的请求。

但是这个方法适用于信息稀疏的情况，试想服务端在短时间内就能收到多个来自四面八方的消息，`http` 连接数无异于 `short polling`，这么做就没啥收益，信息密集型业务更适合使用 `Web Sockets/sse`


在迭代管理&发布平台的前端实现中，我是用轮询不断查询当前迭代部署情况，因为构建部署事件较长，所以每隔 `10s` 查询一次

```ts
function loop() {
  timer = setInterval(() => {
    fetchIterDetail()
  }, 10 * 1000)
}
function deploy() {
  //...
  loop()
}
```

如果整个构建需要 `2min` 完成，那么总共有 `12` 次请求。现在改为 `Long Polling` 优化一下

```ts
// client.ts
function deploy() {
  const res = await Service.startDeploy()
  
  if (res?.data) {
    fetchDetail()
  }
}
async function fetchDetail() {
  const res = await Service.longPolling()
  
  // 异常
  if (res.status != 200) {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    await fetchDetail()
  }
  else {
    // 继续请求，直到部署完成
    if (res.data.msg != 'deploy success') {
      await fetchDetail()
    }
  }
}

// server.js
const Event = require('events')
const messageChange = new Event()

async longPolling(ctx: any) {
  let resolver
  const handler = (data) => {
    resolver(data)
    messageChange.removeListener('change', handler)
  }
  messageChange.on('change', handler)

  // 挂起请求
  await new Promise((resolve) => {
    resolver = resolve
  })
  .then((data) => {
    ctx.body = {
      code: RESPONSE_ENUM.SUCCESS,
      msg: data,
    }
  })
}
function startDeploy() {
  // 模拟构建、部署过程
  const timer = setInterval(() => {
    const num = mock.running()
    if (num >= 12) {
      messageChange.emit('change', 'deploy success')
      clearInterval(timer)
    } else if (num == 6) {
      messageChange.emit('change', 'build success')
    }
  }, 10 * 1000)

  return (ctx.body = {
    code: RESPONSE_ENUM.SUCCESS,
    msg: 'success',
    data: true,
  })
}
```

优化之后，不考虑超时等异常，只有 `2` 次请求。

## SSE

## WebSocket

TODO

## Refer

- [Long polling - javascript.info](https://javascript.info/long-polling)
- [Stream Updates with Server-Sent Events - web.dev](https://web.dev/articles/eventsource-basics)