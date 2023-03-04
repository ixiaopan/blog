---
title: "Event Tracking"
date: "2023-03-04"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---

简单谈一下对埋点的理解

<!--more-->

## What


埋点是对用户在客户端产品发生的事件的描述

- Who：用户

- When：timestamp

- What：用户会有哪些行为？？比如 打开页面、点击按钮、滚动列表、登录、分享等等。

- How：比如 渠道(从哪里访问页面的)、登录方式(email/mobile/sso/...)


## Why

埋点可以帮助业务分析用户行为，从而更好的理解用户习惯、优化产品


## How

### 事件分类


- 页面打开 PageShow

- 用户主动交互 Behavior

一般可以划分为这2个大类，毕竟用户可以只打开页面，什么都不操作

- `PageShow` 需要单独统计，是因为 页面的 `pv/uv` 是埋点中非常重要的一个数据指标

- `Behavior` 可以进一步根据产品的功能再细分，比如产品大概会有 登录、添加购物车、下单、分享等模块


#### 行为属性描述

属性描述，换句话说，就是从多维度分析行为，比如 

- 登录模块，需要统计 登录方式（验证码登录、密码登录）

- 添加购物车，需要统计 商品Id、商家 ID

- 分享，需要统计 分享的商品ID，分享到的渠道


对于页面打开 `PageShow`，就很简单，统计 当前访问的页面ID 即可

### 数据规范/SDK

一条完备的日志需要包含

- 通用数据: `browser/os/screen resolution/appId/appVersion/timestamp/...`

- 页面数据: `pageId/url/query/...`

- 行为数据: `eventType/eventId/extParam/...`

```js

class Tracker {
  page(page?: IPage) {
    const data = {
      type: EVENT_TRACK_TYPE.PAGE_SHOW
    }
    
    this._report(data)
  }

  track(event_id: string, extParam = null) {
    const data = {
      event_id,
      extParam,
      type: EVENT_TRACK_TYPE.BEHAVIOR
    }
    this._report(data)
  }

  private _report(data = null) {
    if (!data) return

    const nextData = {
      common: {
        appId,
        appName,
        appVersion,
        
        os: getOS(), // 系统
        ua: getBrowser(), // 浏览器
        sr: getScreenInfo(), // 屏幕分辨率
      },
      page: {
        pageId,
        title,
        url,
        hostname,
        hash,
        query,
        path,
      },
      event: {
        ...data,
      },
      timestamp: Date.now(),
    }
  }
}
```

### 埋点平台管理

除了 `SDK` 之外，还需要一个埋点平台管理所有埋点，因为开发埋的点位和BI需要统一字段口径，平台就是用来约束埋点值的。


埋点之前，需要去 平台申请对应点位，才能埋点，对应代码就是 `track(event_id: string, extParam = null)` 这里的入参都是埋点平台自动生成的。


如果是开发自己随便写的点位，虽然埋点平台不存在也不会有很大的问题，但是后续和 BI 对口径的时候，就需要去看代码才知道埋的是什么值，就很浪费时间。


实际工作中，埋点平台并没有强约束开发必须要申请，为了不给自己麻烦，尽量还是先申请点位，而且也是为了方便后续的埋点验证工作。

### 埋点验证

最后，为了确定点位确实埋好了，我们需要验证。像是 pc 应用，我们可以直接看 `network` 的请求参数，但是如果是 `h5` 或者是远程调试，是无法直接看到请求的。那该怎么办？

一个方式是建立一个埋点验证平台，访问需要验证埋点的页面就会自动连接到这个验证平台，拦截对应的埋点请求，然后和埋点平台的数据进行匹配即可。

TODO: 等有时间会详细介绍一下怎么实现