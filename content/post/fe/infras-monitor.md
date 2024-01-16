---
title: "性能监控SDK"
date: "2023-12-07"
description: ""
tags: [hands-on]
categories : [
  "Frontend",
]
---

## Background

一开始也没想要做监控，当时其实是接入了阿里的 `arms`，但是接入一段时间后，觉得用起来不舒服，不知道是不是使用姿势不对，而且还收费，因此才有了自己实现一套监控的想法。

功能不需要很多，毕竟产品也没有很多人使用（数量只有百人而已），目的主要是监控异常及时发现问题而非性能优化，并不是说性能不重要，只是当下快速迭代及时发现异常比较重要。

## What

基于这样的想法，要实现的功能就比较明确

- 用户分析
- 行为分析
- JS Error
- API Error
- 链路分析
- 告警


### 用户分析

回答什么人、什么地方、什么时间

- Who: userid
- where: ua, os, geo
- when: time
- bizData：业务数据

```ts
interface IUserData {
  ua?: string
  uav?: string
  os: string
  osVersion: string
  sr: string
  sdkVersion?: string // 小程序基础库
}
```

### 行为分析

做了什么事情 即 behavior

- page url：当前是哪个页面
- click：点击了哪些页面元素

```ts
interface IPageData {
  href: string
}
interface IEventData {
  type: 'click',
  path: string
}
interface IBehaviorData {
  type: 'page' | 'event'
  data: IPageData | IEventData
}
```

### JS Error

分为三类

- JS Error
- Resource Error
- Custom Error
 
```ts
enum ERROR_CATE {
  JS_ERROR = "Js Error",
  CUSTOM_ERROR = "Custom Error",
  RESOURCE_ERROR = "Resource Error",
}

enum LOG_LEVEL {
  INFO = 'info',
  WARN = 'warn',
  ERROR = 'error',
}

export interface IErrorData {
  cate: ERROR_CATE,
  message: string
  stack?: string

  src?: string // resourceError
  nodename?: string
  
  level?: LOG_LEVEL // customError
}
```

### API Error

主要是接口请求异常 以及正常接口的上报

```ts
export interface IHttpData {
  method: string,
  status: string,
  requestTime: number, // 发起请求的时间
  url: string,
  pageUrl: string, // 当前的页面地址
  time: number, // 耗时
  requestParams?: string,
  message: string
  traceId?: string // 后端的traceId
}
```

### 链路分析

根据以上日志，可以串联一个用户在应用内所有的行为轨迹


### 告警

规则定制：
- JS/API 分开配置
- 通用规则
- 某个错误，某个接口，单独定制
- 忽略某些消息

时效性：每隔 `N` 分钟看一下最新数据

阈值：根据规则匹配，发现异常大于 `M` 次

## How

- Step 1 采集数据
- Step 2 分析数据
- Step 3 平台显示


这里重点看下 **Step 1 收集数据** 的设计，数据分析其实就是 mongodb 的聚合查询，不再赘述。


![](/blog/post/images/monitor-sdk.png)


SDK 的设计如上，有2个类 `Collector/Clue`

- `Clue` 对应前面所说的需要统计的数据，目前有7种类型
- `Collector` 则负责管理、采集、发送日志，核心方法就3个
    - `subscribe()` 添加日志采集触发器 `Trigger`
    - `add()` 添加日志
    - `flush()` 发送日志


### Clue

根据上图可知目前有7种日志类型

```ts
enum CLUE_TYPE {
  USER = 'user',
  
  HTTP = 'http',

  BEHAVIOR_PAGE = 'page',
  BEHAVIOR_EVENT = 'event',

  JS_ERROR = 'js_error',
  RESOURCE_ERROR = 'resource_error',
  CUSTOM_ERROR = 'custom_error',
}
```

`clue` 本身不负责发送日志，仅仅表示一条日志，日志本身需要时间戳 + 数据，数据应该由上游处理好之后，传给 `clue`，然后进行进一步的数据扩展、校验等。


```ts
class Clue {
  type: CLUE_TYPE
  timestamp: number
  data: IClueData

  constructor(type, d) {
    this.type = type
    this.timestamp = Date.now()
    this.data = d
  }

  toJSON() {
    return {
      timestamp: this.timestamp,
      ...this.data,
    }
  }
}
```

由此基类可以派生出多种类型日志，鉴于我们这里设计的很简单，不考虑数据扩展、数据校验等，目前的区别不过是 `type、data` 不同而已。为此，为避免冗余，这里采用工厂模式创建日志对象


```ts
class Clue {
  static User(d: IUserData) {
    return new Clue(CLUE_TYPE.USER, d)
  }
  
  static CustomError(d: { message: string, level: string }) {
    return new Clue(CLUE_TYPE.CUSTOM_ERROR, {
      cate: ERROR_CATE.CUSTOM_ERROR,
      ...d,
    })
  }
}
```


### Trigger

日志的数据结构定了，那数据从哪里来呢？这就要配置 `trigger`，即指定什么时候采集数据


上图也可以看到，虽然有 `7` 种类型的日志，但是 `trigger` 只需要 `4` 个，也就是说，一个 `trigger` 可以采集多种类型的日志，是一对多的关系


另外，每个 `trigger` 采集的时机不一样，比如有些是等待网络请求，有些是等待点击事件，不过 `collector` 不必关心时机，只需要把 `add()` 告诉 `trigger`，只要事件一发生，就调用 `add()` 添加到数组


怎么添加一个 `trigger` 呢？ 通过 `subscribe()`


```ts
class Collector {
  add() {}
  subscribe(cb) {
    cb(this.add.bind(this))
  }
}
```

定义一个自己的 `trigger`，比如

```ts
function testTrigger(add) {
  add(new Clue('test_clue', { name: 'test' }))
}
```



#### UserTrigger

首先是 `userData`，一般是用户访问应用就会发送，所以是和初始化 `SDK` 同步

```ts
const userProxy = (add) => {
  add(Clue.User({
    ua: getBrowser(),
    uav: getBrowserVersion(),
    os: getOS(),
    osVersion: getOsVersion(),
    sr: getScreenInfo()
  }))
}
```

#### API Trigger

拦截 `xhr` 的请求和响应，有一个现成的库可以使用 `ajax-hook`，根据文档配置即可

```ts
proxy({
  onResponse(res, handler) {
    add(Clue.Http(re.response), immediate)
    handler.next(response)
  }
})
```

#### Page Trigger

如果是 `SPA`，需要监听路由变化

```ts
window.addEventListener('hashchange', () => {
  add(Clue.EventPage({ href: window.location.href }))
})
```

#### Event Trigger

浏览器中，我们主要监听 `click event`

```ts
function eventProxy(add) {
  window.addEventListener('click', (e) => {
    const data = handle(e)
    add(Clue.EventBehavior(data))
  }, false)
}
```


#### Error Trigger

主要全局监听 `window.error, window.unhandledrejection` 

```ts
window.addEventListener('error', (e: ErrorEvent | Event) => {
  ...
  add(Clue.JSError({ message, stack }))
})
window.addEventListener('unhandledrejection', (e: ErrorEvent | Event) => {
  ...
  add(Clue.JSError({ message, stack }))
})
```



### Flush

最后一个问题，什么时候发送日志？怎么发？

#### 发送策略

1、先看实时发送，只要 `trigger` 触发就上报一条日志（也就是数组长度始终是 `1`)

pros
- 实时，日志不会丢失

cons
- 对于接口密集型应用，会造成网络堵塞


2、如果不实时发送，就要指定规则，比如满足多少条日志量、每隔一段时间上报

props
- 减缓网络堵塞

cons
- 数据不及时，会丢失（当然可以再制定策略避免丢失，比如放在缓存中，不过过于麻烦）


也可以二者结合起来，

- 一旦发生错误，立即上报当前队列所有日志
- 未发生错误之前，正常采集，满足条件才发送


3、虽然制定了上面的策略，但如果条件都没满足，页面被卸载，存储的日志是无法发送的。然而，在页面 `unload` 发送日志，是比较困难的事情，不过也有一些方法可以实现

- 创建一个同步的 `XHR` 即 `xhrReq.open(method, url, true)`
- 通过 `img.src` 发送请求
- 在 `unload` 创建一个 noop 循环


但是这些方法也有一些不足

- `unload` 在某些情况不会触发
- 造成浏览器阻塞，要等请求完成才跳转等


#### sendBeacon

后来 W3C 发布了 [sendBeacon - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator/sendBeacon)，可通过 `POST` 将少量数据 异步非阻塞 传输到服务器。

pros
- 没有跨域问题，所以要注意数据安全
- 请求放在浏览器任务队列，不会阻塞页面卸载等

cons
- 通过 `POST` 只能发送**少量数据**，至于能发送多少数据量，并没有明确
- 只能知道数据是否成功加入队列，不保证成功发送，因此要降级 `xhr`
- 没有 `callback`


#### 怎么发

综上，优先考虑 `sendBeacon` 降级为 `xhr`

```ts
function sendBeacon(url, data) {
  if (navigator?.sendBeacon) {
    return navigator.sendBeacon(url, data)
  }
  return false
}
function sendByXHR() {
  const xhr = new XMLHttpRequest()
  xhr.open('POST', url , true)
  xhr.send(data)
}
function send(url, data) {
  try {
    if (data.length >= 32 * 1024) { // 32KB
      sendByXHR(url, data)
    }
    else if (!sendBeacon(url, data)) {
      sendByXHR(url, data)
    }
  } catch (e) {}
}
```


## Refer
- [sendBeacon](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator/sendBeacon)
- [当 sendBeacon 遇上 Blob
](https://mrluo.life/article/detail/149/send-blob-via-sendbeacon)

