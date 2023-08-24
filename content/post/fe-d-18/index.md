---
title: "D - 基建 API Mock"
date: "2023-08-23"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---


<!--more-->

## Background

开发没有mock还是比较痛苦的，最开始接入mock-server需要在业务里新开一个 `mock` 手动写 json，维护成本很高，也不利于共享。一直以来，都想着接入现在开源的mock产品，但是，一方面需要收钱，一方面比较复杂需要自己搭建服务&数据，一致拖到现在还没有顺手的mock方案。

最近决定自己写个简单的，可能有人说你又在创建轮子，我的想法是，
- 一方面我只开发mock功能，不支持其他比如api自动化测试的功能，我要的功能就是mock
- 另一方面mock通常只是在开发阶段才用到，就那么2天时间，后面进入联调就不会使用了
- 再有就是用来测试接口超时等异常情况

综上，一个简单的mock平台足以满足我的需求，实际上，做这么个平台也就需要1天时间而已。


## Mock平台

### 创建项目

每个业务需要创建对应的项目，我们就简单点


```ts
const MockProjectSchema = new mongoose.Schema({
  id: String,
  name: String,
  desc: String,
})
```

这里 `id` 才是最关键的，后续业务要使用

### 创建 API

`api` 层只要几个功能

- 支持 `GET/POST` 

- 支持超时 `timeout`

- 一个 `json editor`

至于 `api` 的匹配，一期做个模糊匹配即可，也就是只关心 `method/url`，不考虑入参

```ts
const MockAPISchema = new mongoose.Schema({
  projectId: String,
  url: String,
  desc: String,
  method: String,
  mocked: Boolean,
  timeout: Number, // 超时时间
  json: String, // 出参
})
```

然后就是对应的 `crud` 接口实现，比较简单，这里就简单举个例子，不再赘述。

```ts
class MockMeController {
  prefix = '/mockme'

  async queryProjectById(ctx: any) {
    const { id } = ctx.request.params
    const data = await MockProjectModel.findOne({ id }, { __v: 0, _id: 0 })
    return data
  }
}
```


## 业务接入

重点看看，业务怎么接入平台


### mock option

从开发者的角度思考，他只需要在 `api` 这个文件对需要 mock 的 api，加一个参数即可

```ts
async queryList() {
  await request.get({
    url: 'test/list',
  }, {
    mock: true, // 开启 mock
  })
}
```

要想拿到 mock 数据，必须和 mock 平台关联，也就是要拦截请求，代理到平台的地址。以 `axios` 为例，我们封装如下方法


```ts
async get(confg, option) {
  return await request({ ...conf, method: 'get', }, option)
}
async request(conf, option) {
  // 判断接口是否开启了 mock
  config.url = option.mock ? `${mockUrl}${config.url}` : conf.url
  await axios.request(conf)
}
```


这段代码判断是否开启了 mock，如果开启，则把 `mockUrl` 和业务的 `url` 拼接起来，这里的 `mockUrl` 是代理到平台的地址，这样就实现了请求的拦截。

不过，这里会有跨域的问题，后面会解决这个问题


### mock url

`mockUrl` 是 `mock` 平台提供的，前面我们说过每个业务对应一个mock项目，也就会有对应唯一的项目id，平台根据这个 id会自动生成一个地址，格式如下

```json
https://test.com/mockme/proxy/:projectId
```

- `test.com` 是平台服务器地址

- `/mockme/proxy`是路由

- `projectId` 就是每个业务在 mock 平台的项目 id


举个例子

```ts
// 项目一
https://test.com/mockme/proxy/elem

// 项目二
https://test.com/mockme/proxy/ant-design
```

请求打到平台后，平台会有对应的接口处理


```ts
class MockMeController {
  ...
  async proxyAPI(ctx: any) {
    const urlRegResult = ctx.request.path?.match(/\/mockme\/proxy(.*)/)
    const [, regUrl] = urlRegResult
    const pathNode = pathToRegexp('/:projectId/:proxyURL*').exec(regUrl)
    const [, projectId, proxyURL] = pathNode

    const doc = await MockAPIModel.findOne({ projectId, url: '/' + proxyURL })
  }
}

router.all(`/mockme/proxy/:id/(.*)`, proxyAPI)
```

这里有几点注意

- 我们使用 `router.all` 来覆盖所有的 `http method`，毕竟不知道业务会有哪些方法

- 通过 `path-to-regexp` 转换为 url 为正则，方便匹配需要的数据



### 跨域

Q：前面说过代理地址会有跨域问题，那该怎么解决呢？

A：mock 平台配置 `cors` 


Q：但是怎么识别请求是真的来自业务呢？`cors` 只能对白名单列表的 `url` 开启，

A：业务连接到 mock 平台，可以带个请求头，表明身份


以 `axios` 为例，只要添加一个 `requestInterceptor` 即可

```ts
this.axiosInstance.interceptors.request.use((config) => {
    // 加签校验，配置 mock 请求头
    if (options?.serverMode && options?.mock) {
      config.headers['x-test-mock'] = 'mock-me'
    }
    return config
}, undefined)
```

服务端则检测请求头，进行白名单校验，以 `koa` 为例

```ts
app.use(
    cors({
      origin: (ctx: any) => {
        // cors 预检
        if (ctx.header['access-control-request-headers']?.includes('x-test-mock')) {
          return url
        }

        if (ctx.header['x-test-mock'] == 'mock-me') {
          return url
        }
      },
      allowHeaders: ['Content-Type', 'x-test-mock'],
    })
  )
```

综上，一个简单的 api mock 平台就开发好了，还有更多功能可以扩展，比如 `api` 自动化测试、实现网页版本 `postman` 等。