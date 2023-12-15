---
title: "基建 埋点管理"
date: "2023-08-22"
description: ""
categories: [
  "Frontend",
]
---

简单实现一个埋点sdk和管理平台

<!--more-->

## What


埋点是对用户在客户端产品发生的事件的描述

- Who：用户

- When：timestamp

- What：用户会有哪些行为？？比如 打开页面、点击按钮、滚动列表、登录、分享等等

- How：比如 渠道(从哪里访问页面的)、登录方式(email/mobile/sso/...)


## Why

埋点可以帮助业务分析用户行为，从而更好的理解用户习惯、优化产品


## How

### 事件分类

一般可以划分为2个大类

- 页面打开 `PageShow`，需要单独统计，因为页面的 `pv/uv` 是埋点中非常重要的一个数据指标

- 用户主动交互 `Behavior`
  - `click` 点击行为
  - `exposure` 模块的曝光

```ts
enum IEvent_Type = {
  PAGE_SHOW = 'pageShow',
  CLICK = 'behavior.click',
  EXPOSURE = 'behavior.exposure',
}
```

### 数据规范

#### `pageShow` 描述

对于页面打开 `PageShow`，就很简单，一般统计 当前页面 `url/pageId` 即可


#### `behavior` 描述

试想怎么描述 『点击了登录按钮』这件事件，至少需要


```ts
interface IEvent {
  eventId: string, //  行为 `id`，比如下载download, 分享 share, 登录 login
  eventType: IEvent_Type,
  eventTarget: IEventTarget[], // 事件对象的描述，类似 `js` 里的 `event.target`
  timestamp: number, // 发生时间
}
```

#### 事件对象的描述

事件对象的相关信息，比如 

- 点击登录，需要统计 登录方式（验证码登录、密码登录）

- 添加购物车，需要统计 商品Id、商家 id

- 分享商品，需要统计 分享的商品id，分享渠道



举个例子，分享商品

```ts
{
  eventId: 'share',
  eventType: IEvent_Type.CLICK,
  eventTarget: [
    {
      goodId: '1',
      channel: 'wechat',
    },
    {
      goodId: '2',
      channel: 'wechat',
    }
  ],
  timestamp: Date.now()
}
```

上面这个 `eventTarget` 还是有一点点问题，不方便后续的数据分析，上面的例子中事件对象是 `goodId`，换到其他业务，这个对象可能是 `projectId`，换句话说，`evetTarget` 的结构不固定，无法统一分析。优化后的结构如下，其中 `extend` 作为扩展字段，存储其他业务信息，比如说上面的渠道

```ts
interface IEventTarget {
  targetKey: string
  targetValue: string
  extend?: { [k: string]: any }
}
```

```ts
{
  eventId: 'share',
  eventType: IEvent_Type.CLICK,
  eventTarget: [
    {
      targetKey: 'goodId',
      targetValue: '1'
      extend: {
        channel: 'wechat',
      }
    },
    {
      targetKey: 'goodId',
      targetValue: '2'
      extend: {
        channel: 'wechat',
      }
    }
  ],
  timestamp: Date.now()
}
```

#### 上下文环境

除了事件本身，还需要知道事件发生时所处的环境，

- 设备信息

- 业务应用信息

- 用户信息

- 页面信息



综上，我们可以定义一条完备的日志数据结构


```ts
interface ITrack {
  common: {
    appId: string // 应用ID
    appName: string, // 应用名称
    appVersion: string // 应用版本
        
    os: string, // 系统
    ua: string, // 浏览器
    sr: string, // 屏幕分辨率
  },
  page: {
    url: string,
    // pageId: string, // 只要能区分page即可
  },
  user: {
    userId?: string
  },
  event?: IEvent,
  timestamp: Number, // 发送日志的时间
}
```

### 埋点平台管理

除了 `SDK` 之外，还需要一个埋点平台管理所有埋点，因为开发需要和 `BI` 统一数据口径，而平台就是用来约束埋点值的。


具体来说，开发埋点之前，需要去 平台申请对应点位，然后写在业务里。

实际工作中，埋点平台并没有强约束开发必须要申请，但是如果开发随便写点位，没有去平台申请，后续和 BI 对字段的时候，需要去看代码才知道埋的是什么值，这就很浪费时间。


为了不给自己麻烦，尽量还是先申请点位，而且也是为了方便后续的埋点验证工作。

### 埋点验证

最后，为了确定点位确实埋好了，我们需要验证。一般是看 `network` 的请求参数，不过这个方式有几个缺点

- 业务请求太多，难以清晰的找到埋点的接口，需要开启过滤

- 一般而言， `h5` 非模拟器环境是无法直接看到请求的

针对以上问题，可以考虑2种小工具提高验证效率

- `SDK` 内置 `UI` 查看工具，自动显示每次的埋点数据

- 打通埋点平台，拦截业务发起的埋点请求，对其进行数据结构的校验


## 技术细节

### 埋点方式


埋点有可视化埋点、自动埋点、手动埋点，各有利弊

- 可视化埋点

- 自动埋点

- 手动埋点

我们业务比较简单，就采用了手动埋点


### 发送时机

上面我们已经确定了埋点的数据结构，接下来要考虑什么时候发出去，一般有2种方式

- 实时发送，简单，事件发生就发生一条日志
  - 对接口密集型业务，会影响业务的请求
  - 几乎不存在数据丢失的情况

- 延时发送，比较复杂，需要考虑很多东西，比如
  - 发送时机，比如页面切换 或者 定时器机制；
  - 未来得及发送的数据进行缓存，下次进入页面再发送；
  - 比较容易存在数据丢失的情况


### 怎么发送

发送埋点本质就是发送一条请求，可能大家就说那不简单，`xhr` 不就得了，但是，有几个问题需要考虑

- 发送时机，要知道如果在页面离开发送请求，一般浏览器会中断该请求，除非某些浏览通过代码配置允许不中断请求

- `xhr` 会有跨域问题



#### XHR


#### img


#### sendBeacon


### SDK

#### 初始化

根据前面的数据结构定义，需要外部传入的参数只有应用元信息、用户身份

```ts
type IUser = {
  userId: string
}

type IOption = {
  appId: string,
  appName: string,
  appVersion: string
  
  userInfo: () => IUser
}
```

在SDK初始化的时候，会做几件事情

- 初始化上下文信息

- 页面信息管理

- 注册插件系统，在埋点最后上报之前，提供给业务进行数据处理的钩子
  - SDK 内置了一个 `validate` 插件，会连接到埋点平台自动验证


```ts
class MyTracker {
  private common
  private pageStack: MyPage
  private _transform

  constructor(opts: IOption) {
    const { appId, appName, appVersion } = options || {}
    
    this.common = {
      os: getOS(), // 系统
      ua: getBrowser(), // 浏览器
      sr: getScreen(), // 屏幕分辨率
      
      appId,
      appName,
      appVersion,
    }
 
    this.pageStack = new MyPage() 
    
    // 插件注册
    const plugins = (options?.plugins || [])
    if (options?.validate) {
      plugins.unshift(ValidatePlugin)
    }
    this._transform = compose(plugins)
  }
}
```

`MyPage` 负责页面信息管理，默认只保留当前访问的页面信息

```ts
class MyPage {
  private history: { url: string }[]
  private maxLength
  constructor(opts?) {
    this.maxLength = opts?.maxLength || 1
  }
  record() {
    this.history.push({ url: location.href })
    this.history.slice(-1 * this.maxLength)
  }
  last() {
    return this.history[this.history.length - 1]
  }
}
```

最后，暴露2个上报方法给业务

```ts
// 上报pageShow
page(page?: IPage, params = null) {
  this.pageStack.record(page)
  this._report({
    type: IEvent_TYPE.PAGE_SHOW,
  })
}

// 上报behavior
track(log: IEvent) {
  this._report({
    type: IEvent_TYPE.CLICK,
    log,
  })
}

```

### 插件系统

埋点发送之前，会经过一系列注册的插件，类似 `axios` 的 `interceptors`，一个顺序执行的插件流

```ts
export function compose(plugins) {
  if (!Array.isArray(plugins)) {
    throw new Error('plugins must be an array')
  }

  if (plugins?.some(p => !isFunction(p))) {
    throw new Error('plugins must be all functions')
  }

  return (ctx, initialData, doneCallback) => {
    let lastIndex = -1
    
    const step = async (index: number, data: any) => {
      if (index == lastIndex) {
        throw new Error('next() called multiple times')
      }

      if (index == plugins.length) {
        return doneCallback(data)
      }
      
      lastIndex = index
      
      const callback = plugins[index]
      await callback(ctx, data, (nextData: any) => {
        step(index + 1, nextData)
      })
    }

    step(0, initialData)
  }
}
```

前面说明，埋点验证有2个小工具，一个是UI查看工具，一个是自动验证，本质就是2个插件


UI查看工具的话，我们是封装一个 `web component`，业务开启的话，会在业务页面弹出一个浮层，每次埋点上报都能看到埋点数据

```ts
class TrackDebug extend HTMLElement {
  container

  constructor() {
    super()
    
    const shadowRoot = this.attachShadow({ mode: 'open' })
    shadowRoot.appendChild(this.template())
    this.container = shadowRoot.querySelector('#track-list')
  }
  private template() {
    const template = document.createElement('template')
    template.innerHTML = `<ul id="track-list"></ul>`
    return template.content.cloneNode(true)
  }

  static get observedAttributes() {
    return ['data-list']
  }
  attributeChangedCallback() {
    this.render()
  }
  addLog(data: {
    log: string
    timestamp: string
  }) {
    const list = this.list || []

    list.unshift(data)

    this.setAttribute('data-list', JSON.stringify(list))
  }
  render() {
    let data = []
    try {
      data = JSON.parse(this.getAttribute('data-list'))
    } catch (e) {}

    const liStr = data.map((item: { log: string; timestamp: string; }) => {
      return `
        <li>
          <p>${item.log}</p>
          <span>${formatTime(item.timestamp)}</span>
        </li>`
      }).join('')
    
    this.container.innerHTML = liStr
  }
}
customElements.define('track-debug', TrackDebug)
```

然后注册一个插件，每次埋点发送都会经过这里，从而添加到 `UI` 上即可

```ts
let trackIns
export default async (ctx, data, next) => {
  if (!trackIns) {
    trackIns = document.createElement('track-debug')
    document.body.appendChild(trackIns)
  }
  trackIns?.addLog(data)
  next()
}
```

同理，自动验证的话，只需要连接到埋点平台的验证接口，不合规的埋点不会发送请求

```ts
export default async (ctx, data, next) => {
  // 数据校验
  let valid = true
  if (ctx.options.validate) {
    try {
      const res = ctx.options.validate ? await checkValid(data) : await Promise.resolve({ data: true })
      valid = !!res?.data
    } catch (e) {}
  }
  
  // 校验成功，才执行下一个插件
  if (!ctx.options.validate || valid) {
    next(data)
  }
}
```

最后，插件流转完成，会回到最后的发送接口

```ts
private _report(data: { type: IEvent_Type, log?: IEvent }) {
  const logData: ITrack = {
    common: this.common,
    user: this.options?.userInfo()
    page: this.pageStack.last,
    timestamp: Date.now(),
    event: log,
  }

  // 插件处理
  this._transform(this, logData, (finalData) => {
    sendImmediateLog(finalData)
  })
}
```


## SPM

一些公司采用的埋点方案是 `spm`，数据形式类似 `siteA.pageB.blockC.targetD`，有4个点位信息

- a 是应用 id

- b 是页面 id

- c 是区块 id

- d 是事件 id

这些 `id` 都是在平台申请的，申请下来的 `id` 也是诸如数字的形式，比如，我们有应用 `saas` 页面是详情页，发生了一次点击分享行为，这个点位是 `a101.b20.c32.d12`。

这个方案相比较前面的方案还是比较复杂的

- 埋点值不是很直观，点位都是数字，很难理解

- `SDK` 需要支持自动埋点，不然开发面对这些数字要头大

- 需要完善的埋点平台的支持（怎么申请点位、点位的管理、点位的校验、数据分析等）


这套方案之所以这么复杂，应该是为了满足淘宝的需求，店铺、商品详情页，需要统计各个模块、展位的曝光率、点击率等。
