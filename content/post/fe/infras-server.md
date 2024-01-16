---
title: "服务端实现"
date: "2024-01-02"
description: ""
tags: [hands-on]
categories: [
  "Frontend",
]
---


## Background

前面一系列关于基建的文章主要侧重子模块的实现，但是技术选型还没有确定，此外虽然模块功能已经确定，但还缺少服务端必不可少的基础服务，比如日志、鉴权等。

`Node` 端比较流行的服务框架有 `koa/express/Nest/Nuxt` 等等，鉴于我们主要是实现简单的 `http endpoint`，使用 `koa/express` 就足以，本文以 `koa` 为例，数据库选择 `MongoDB/Redis` 更符合前端人员的开发思维。


## 登录

既然是内部工具，一方面是保证系统安全性不被外人访问到，一方面就是免登，最简单的方案就是接入内部使用的 `IM`，比如钉钉、飞书等。

### 飞书登录流程

这里我们以飞书为例，流程参考 [飞书登录流程](https://open.feishu.cn/document/common-capabilities/sso/web-application-sso/web-app-overview)，其实很简单，授权登录是一个拿 `code` 换 `token` 的流程。从前端开发的角度来看，需要调用2个 `API`

1、获取 code

```ts
async auth(ctx: any) {
  const { redirect_uri } = ctx.request.query || {}
  const query = `client_id=${client_id}&redirect_uri=${encodeURIComponent(redirect_uri)}`
  return `https://passport.feishu.cn/suite/passport/oauth/authorize?${query}`
}
```

服务端会返回如上的 `url` ，客户端拿到后手动跳转，用户授权之后，飞书会自动在 `redirect_uri` 中加入 `code`

2、前端从地址栏解析 `code`，再次调用后端接口获取 `token` 然后据此获取用户信息

```ts
class AccountController {
  async token(ctx: any) {
    try {
      let res = await axios.post('https://passport.feishu.cn/suite/passport/oauth/token', {
        grant_type: 'authorization_code',
        client_id,
        client_secret,
        code, // 地址栏解析的 code 
        redirect_uri,
      })

      // 拿到 token 获取用户信息
      const { access_token } = res.data
      res = await axios.get('https://passport.feishu.cn/suite/passport/oauth/userinfo', {
        headers: {
          Authorization: `Bearer ${access_token}`,
        },
      })
      const { user_id, name, avatar_url } = res.data || {}
    } catch (e) {
      logger.error(e)
    }
  }
}
```

到此，前后端都知道用户身份了，不过前后端保存用户身份的逻辑不同

1、前端 把 `token` 写入到 `localStorage` 中，后续的接口调用都要在 `header` 带上它再传给后端验证

2、后端 写入 `mongodb`，不过通常 `token` 具有有效期，写入 `redis` 可以快速查询是否过期，

```ts
let doc = await AccountModel.findOne({ userId: user_id })
if (doc) { // 同一个用户，更新 token 而已
  doc.token = access_token
  const res = await doc.save()
} 
else { // 说明是新建的
  await AccountModel.create({
    name,
    userId: user_id,
    avatar: avatar_url,
    token: access_token,
  })
}
```

### 永久登录

在我们这个内部系统，为了避免老是登录，就做了一个假的永久登录态，即使 `token` 失效也不管。换句话说，

1、只要飞书授权成功之后，就把 `token` 更新到数据库中（见上面的代码），不在考虑 `refreshToken` 的事情

2、鉴权只判断数据库中是否存在 `token`，存在即有效，不存在就失效

3、除非用户主动退出，才会清空数据库中的 `token`


这个方案简单粗暴，有个很明显的问题，如果用户在不同的端访问系统，`token` 会被反复覆盖。举个例子，

1、用户先用PC端浏览器访问，因为是第一次访问，会生成 `tokenA` 写入数据库，

2、再用 pad 端浏览器访问，因为是第一次访问，又会生成新的 `tokenB` 从而覆盖 `tokenA`， 因为此时数据库中已经有 `user_id` 了，这时候更新 `token` 即可，而不是创建一个新用户

3、过一段时间，再用PC访问，此时是二次访问，`headers` 中带有 `tokenA`，但是数据库已经找不到 `tokenA` 从而强制用户退登

解决方法也很简单，加个 `redis` 既支持真的 `token` 有效期，也支持多端登录。

## koa 中间件

### API 鉴权

大多数 `API` 都是要鉴权的，不过也有少部分不需要，比如前文提到的2个接口 `auth() & token()`，此时还未授权，哪来的 `token`。代码参考如下

```ts
const whitePathList = [
  '/account/auth',
  '/account/token',
]
export function tokenIntercept() {
  return async (ctx, next) => {
    if (whitePathList.includes(ctx.request.path)) {
      return next()
    }
    
    const tokenValue = ctx.request.headers['x-access-token']
    
    if (!tokenValue) {
      return 'token is null'
    }
    
    const user = await AccountModel.findOne({ token: tokenValue })
    if (!user) {
      return 'invalid token'
    }

    await next()
  }
}
```

### 跨域

即使通过白名单跳过 `token` 验证，某些 `api` 依旧访问不通，会出现跨域的问题，比如埋点系统的自动验证插件，会调用服务端的 `/track/validate`、[mock 平台](/blog/post/fe/infras-mock/#%E8%B7%A8%E5%9F%9F)的 `/mockme/proxy`。


跨域也很好解决，最简单的就是服务端支持 `cors`，koa 有自己的三方中间件。这里从3个方面判断是否可以跨域

- 域名是否在白名单内
- 请求头 `header` 
- 请求方法

```ts
import cors from 'koa2-cors'
app.use(
    cors({
      origin: (ctx: any) => {
        const url = ctx.header.referer?.substr(0, ctx.header.referer.length - 1)
        if (url && whiteList.some((u) => url.includes(u))) {
          return url
        }
        if (ctx.header['x-test-mock'] == 'mock-me') {
          return url
        }
      },
      allowMethods: ['POST', 'GET'],
      allowHeaders: ['Content-Type', 'x-access-token', 'x-test-mock'],
    })
  )
```

### log4js
最后就是日志服务，服务端不像前端，可以到直接在浏览器 `debug`，所以后端的日志服务是非常重要的，这里选择了比较流行的一个三方日志库 [log4js-node](https://github.com/log4js-node/log4js-node)，功能挺多的。

那我们要搜集哪些日志信息呢？？我分为3大类
- 所有日志：info、warning、error 所有信息，帮助分析用户行为
- 错误日志：代码异常、数据库异常等
- API日志：请求了哪些接口？接口的耗时分布？


```ts
log4js.configure({
  appenders: {
    console: {
      type: 'console',
    },

    // 所有的日志
    app: {
      type: 'file',
      filename: 'log/app.log',
      maxLogSize: 10485760,
    },

    // 访问接口日志
    access: {
      type: 'dateFile',
      filename: 'log/access.log',
      pattern: 'yyyy-MM-dd.log',
      alwaysIncludePattern: true,
    },

    // 错误日志
    error: {
      type: 'dateFile',
      filename: 'log/error.log',
      pattern: 'yyyy-MM-dd.log',
      alwaysIncludePattern: true,
    },
    'just-error': {
      type: 'logLevelFilter',
      appender: 'error',
      level: 'error',
    },
  }
})
```

这段代码的 `error appender` 使用了 [logLevelFilter](https://log4js-node.github.io/log4js-node/logLevelFilter.html)，简单来说就是根据 `level` 过滤事件。比如线上环境，我们希望 `info` 以上的事件记录到 `app appender` 中同时 `error` 以上的事件记录到 `error.log` 中

```ts
log4js.configure({
  // ...
  
  categories: {
    default: {
      appenders: ['just-error', 'app'],
      level: 'info',
    },
    access: {
      appenders: ['access'],
      level: 'info',
    },
  }
})

export let logger = log4js.getLogger()
```

#### API 日志

要想记录访问了哪些api、入参等信息，显然需要实现中间件，代码也很简单


```ts
const logAccess = () => {
  const accessLogger = log4js.getLogger('access')

  return async (ctx, next) => {
    accessLogger.info(
      JSON.stringify({
        url: ctx.url,
        query: ctx.query,
        ua: ctx.userAgent,
        timestamp: Date.now(),
      })
    )

    await next()
  }
}
```


## 待改进

目前的服务端是所有子项目、定时任务部署在一起

1、只要其中一个服务挂起，其他服务也会受到影响

2、只改某个项目的代码，也会部署整套服务

只因现在业务不复杂，这样做是可以的，如果后续继续完善各个项目，单独部署是有必要的，后续可以考虑 `nestjs` 以及微服务。 