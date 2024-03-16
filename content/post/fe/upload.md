---
title: "Upload"
date: "2023-12-16"
categories: [
  "Frontend",
]
---

## Background

业务中涉及大量上传，应该说上传是业务的基石，因此只有理解了上传的逻辑，才能服务好业务。本文主要从以下几个方面总结关于上传的开发经验

- 使用OSS上传
- 多个大文件同时上传
- 从0到1实现上传


## 使用OSS上传

业务中使用的是阿里云OSS，就以此为例说一下如何使用已有的SDK完成业务，核心流程如下图

![](/blog/post/images/oss.jpg)

### STS

首先 `OSS` 采用临时 `STS` 进行授权访问，所以第一步要有获取到凭证才能进行上传。这一步可以参考官方文档 [Node.js授权访问 - Ali OSS](https://help.aliyun.com/document_detail/32077.html)，具体代码参考如下

```ts
const OSS = require('ali-oss')
export async function queryOssSTS() {
  const sts = new OSS.STS({
    accessKeyId: ACCESS_KEY_ID,
    accessKeySecret: ACCESS_KEY_SECRET,
  })
  try {
    const res = await sts.assumeRole(ACCESS_ROLE, ``, '1800', 'sessiontest')

    return {
      AccessKeyId: res.credentials.AccessKeyId,
      AccessKeySecret: res.credentials.AccessKeySecret,
      SecurityToken: res.credentials.SecurityToken,
      Expiration: res.credentials.Expiration,
      Endpoint: END_POINT,
      bucketName: BUCKET_NAME,
      host: 'https://xxxx.oss-cn-hangzhou.aliyuncs.com',
    }
  } catch (e) {
    logger.error(e)
  }
}
```

### 初始化

前端获得凭证后，就可以初始化OSS实例

```ts
client = new OSS({
  accessKeyId: res.AccessKeyId,
  accessKeySecret: res.AccessKeySecret,
  stsToken: res.SecurityToken,
  region: 'oss-cn-hangzhou',
  bucket: res.bucketName,
  endpoint: res.Endpoint,
  refreshSTSToken: async () => {
    const refreshToken = await getSTS()
    return {
      accessKeyId: refreshToken.AccessKeyId,
      accessKeySecret: refreshToken.AccessKeySecret,
      stsToken: refreshToken.SecurityToken,
    }
  },
})
```

其中 `res` 就是前文 `node` 服务返回的结果，要注意既然是临时`token`，所以就有过期的时候，过期可以通过`refreshSTSToken`拿到新token就行。


### 简单/分片上传

最后就是上传文件，对于大文件建议走分片上传，这样做不仅可以提高上传速度，还可以实现断点续传、获取到上传进度等

```ts
// 大于100M
if (file.size > 100 * 1024 * 1024) {
  client.multipartUpload(filePath, file, {
    parallel: 4,
    partSize: 1024 * 1024, // 1MB
    progress: (p: number, cpt: any) => {
    // 获取上传进度
    console.log(Math.floor(p * 100) + '%')
    },
  ).then(() => {})
} else {
  client.put(filePath, file).then(async (result: { url: string }) => {})
}
```

### 简单上传模拟进度条

根据官方文档，简单上传不支持获取进度的，没办法如果非要显示进度只能想办法模拟。一个思路是获取当前网速

```ts
let bytePerSec
navigator.connection.addEventListener('change', () => {
  bytePerSec = (navigator.connection.downlink * 1024 * 1024) / 8
})
```

然后再计算时差乘以网速得到网络包大小从而模拟进度，需要注意 `clearInterval` 避免内存泄露

```ts
let timer
simpleUpload() {
  timer = setInterval(() => {
    const fakeSize = bytePerSec * (Date.now() - startTime) / 1024

    // 因为是模拟进度条，不能完全给100%
    const fakeProgress = Math.min(99, fakeSize / fileSize * 100)
    this.onProgress(fakeProgress)
    
    // 模拟到头了。。。取消定时器
    if (fakeProgress == 99) {
      clearInterval(timer)
    }
  }, 100)

  // 无论失败成功，都取消定时器
  this.put().then().finally(() => clearInterval(timer))
}
```

### 封装成class

可以把上面的代码封装一下，既提供工厂模式，也支持单例模式

```ts
// utils/oss.ts
export class MyOSS {
  constructor() {
    const client = new OSS({})
  }
  autoUpload(resource: IResource) {}
  // ...
  static create() {}
}
export default MyOSS.create()
```


## Resource

对于业务要上传的文件，我们抽象为一个对象 `resource`，至少包括要上传的文件本身，此外还可以配置几个事件钩子

```ts
interface IResource {
  file: File
  id?: string
  name?: string
  bizData?: { [k: string]: any }
  onBeforeLoad?(): void
  onProgress?(percent: number): void
  onComplete?(data: { url }): void
  onError?(error: string | Error): void
}
```

封装之后，对于业务来说就很简单了，实际上，前文基建中提到的文件上传平台就是这么实现的。

```ts
// 业务使用
import myOSS from 'utils/oss'
myOSS.autoUpload({ file, onComplete() {}, onError() {}, })
```

## 多个大文件同时上传

如果只是单独的上传一个文件，以上代码都足以应付，但真正的业务远不可能这么简单。在我们的业务中，上传是业务的基石，上传是使用系统的第一步，文件进入系统后才能有后续的管理、权限等。如此，就不可能让用户一个一个上传文件，相反，要支持多文件、文件夹同时上传。

### 文件状态

第一个问题，UI层要感知文件状态，状态流转也很简单，见下图

![](/blog/post/images/upload-status.jpg)


### 中间件

第二个问题，每个文件在**上传开始前**可能会有一些业务逻辑，比如下图中的 `addRecord` 用来更新上传记录、`updateMD5`进行 `md5` 校验等。

此外，不同入口下的上传会有不同的业务逻辑，下图有3个入口，每个入口要处理的业务逻辑是不一样的。

![](/blog/post/images/upload-midware.png)

很容易想到使用中间件来解决这个问题，中间件也有很多模式，针对该业务逻辑，使用最简单的顺序型模型就足以，参考代码如下

```ts
class Sequence {
  constructor() {
    this.tasks = []
  }
  push(callback) {
    this.tasks.push(callback)
  }
  async run(context: any, doneCallback) {
    let lastIndex = -1
    const step = async (index: number, data: any) => {
      if (index == lastIndex) {
        throw new Error('next() called multiple times')
      }
      if (index == this.tasks.length) {
        return doneCallback(data)
      }
      lastIndex = index
      const callback = this.tasks[index]
      if (callback) {
        await callback(context, (data: any) => {
          step(index + 1, data)
        })
      }
    }
    await step(0, {})
  }
}
```

### 跳过OSS上传
上传并不一定总是来自前端，也可能是服务端上传，前端不断轮询对应文件状态，比如上图中的 `entry 1` 这种情况。这种要怎么跳过前端 `oss` 上传同时又能更新状态呢？


1、所有中间件执行完成后的方法中，判断是否跳过 `oss`


上图可以看到，所有中间件执行完成后，会进入到一个 `doneCallback` 中，这里会开始真正的上传文件，最终会调用 `oss.autoUpload()`，所以这里加个判断是否需要前端上传，字段 `noClientOSS` 当然是业务指定了

```ts
resource.sequence.run(resource, () => {
  if (!resource?.bizData?.noClientOSS) {
    oss.autoUpload()
  }
})
```

2、轮询接口中，更新 `resource`

前文已经说过，我们把文件抽象成一个 `IResource`，有自己的状态、事件钩子，同时轮询是其中一个中间件，所以我们可以在其中拿到 `resource`

```ts
function pollMiddleware(resource, next) {
  function queryProgress() {
    const data = await api.queryProgress()

    if (data.status == 'DONE') {
      resource.onComplete(data.url)
      next()
    }
    else if (data.status == 'ERROR') {
      resource.onError()
      next()
    }
    else {
      resource.onProgress(data.progress)
    }
  }
  setInterval(queryProgress, 2000)
}
```

### 并发
最后一个问题，浏览器同一时间内支持的 `http` 并发数量是有限制的，比如 `chrome` 可能是 `6-10` 个，现在我们上传了1000个文件，同时又是分片，如果每次分片数量是4个，那就是同一时间内有4000个 `http` 发送出去，就会造成网络堵塞，甚至浏览器崩溃。所以，我们要做一个队列，同时间只能 `N` 个文件在上传，也就是并发控制的实现

```ts
function concurrency(urlList, n) {

}
```

## ResourceManager

从另外一个角度来看，每个文件都要在UI层更新自己的状态，为了避免业务直接接触到每个文件，我们还要实现一个中心管理器 `ResourceManager`，文件的动态统一收口在管理中心，同时管理中心还可以实现并发的调度。


综上，最后的架构图如下

![](/blog/post/images/resource-manager.png)


## From Scratch


## Reference

- [Node.js授权访问 - Ali OSS](https://help.aliyun.com/document_detail/32077.html)
- [Browser.js上传文件概述 - Ali OSS](https://help.aliyun.com/zh/oss/developer-reference/overview-2)
- [Network information](https://googlechrome.github.io/samples/network-information/)
