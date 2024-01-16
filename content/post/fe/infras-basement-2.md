---
title: "迭代管理&发布平台 02"
date: "2023-12-30"
description: ""
tags: [hands-on]
categories: [
  "Frontend",
]
---

## Background

之前介绍了开发这个平台的初衷以及平台具有的功能，是因为当下工作流太割裂
- 需求/技术文档在飞书，但不知道该需求对应哪个分支
- 代码托管在 `gitlab`，但不知道该分支是需求分支、还是个人开发分支
- 部署在另外的平台比如云效，只看分支名压根不知道是什么需求

上面3个平台单独来看，根本无法知道在做什么需求，需求的进度如何等。比如要查看前一周上线了什么需求，就只能查看 `git` 提交历史，或者部署记录，但如果部署分支是 dev/release/main，那根本不可能知道上线了什么内容。

那这个平台的目的是集成以上3者的功能，让迭代有迹可循，让开发者更关注开发，让项目管理一目了然。

从用户角度来看，当进入开发阶段，只需要打开平台
- 新建一个迭代，关联相关需求、技术文档
- 提交代码，在平台部署到对应环境
- 在平台申请发布，然后上线

## 应用

以前新立项一个项目，是在 `gitlab` 上创建；现在，到平台创建一个应用。


### 关联 Git

一个应用除了基本的信息（应用名、描述、维护者等）之外，最关键的是要关联对应的 `git` 项目上，目的是获取 `git project id`，为后续的分支管理、`MR` 等准备。

此时，应用的 `schema` 可以定义如下

```ts
interface IUser {
  name: string
  userId: string
}
enum APP_TYPE {
  WEB = 'web',
  H5 = 'h5',
  MINI_PROGRAM = 'mini-program',
  HYBRID = 'hybrid',
  APP = 'APP',
}
interface BasementSchema {
  projectId: string // gitlab project id
  name: string // gitlab project name
  url: string // gitlab project url
  desc: string // 应用描述
  type: APP_TYPE  // 应用类型 
  maintainer: IUser[] // 维护者
}
```

### 项目模板

基本上每个前端团队都有自己的项目脚手架 `cli`，一键生成项目，保证前端基建统一。

既然现在有了平台，我们就把 `cli` 接入平台，在创建新项目的时候，选择对应项目模板，相应的，应用 `schema` 此时需要新增一个模板字段

```ts
enum TEMPLATE_TYPE {
  BLANK = 'blank'
  VUE = 'vue',
  REACT = 'react',
  H5 = 'h5',
  NPM = 'npm',
  VITE_PLUGIN = 'vite-plugin',
}
interface BasementSchema {
  // ...
  template: TEMPLATE_TYPE // 项目模板
}
```

### monorepo

这里要考虑当下 `monorepo` 的项目架构，最开始开发的时候我就没考虑到，不过后续扩展也简单，因为这里主要是维护**项目元信息**。

对于 `monorepo` 而言是共享 `git project id`，不同在于项目名，所以新增一个字段 `aliasName` 支持自定义业务别名，即使不是 `monorepo` 也是可以使用的。对于 `monorepo` 的项目当然要加个字段标识进行区分

```ts
interface BasementSchema {
  // ...
  aliasName: string // 项目别名
  monorepo: boolean
}
```

### 分支规范/MR管理

只要有了 `git project id`，就可以对分支、MR进行增删查改，`API` 接入参考 

- [Gitlab API](https://docs.gitlab.com/ee/api/merge_requests.html)
- [Ali Codeup API](https://help.aliyun.com/document_detail/460464.html)

`gitlab` 上的分支列表是没有业务属性的，但是我们内部约定

- `story/*` 属于需求分支，同时也是保护分支，个人开发只能提交MR到此分支
- `refactor/*` 属于技改分支，
- `feat/*` 个人开发分支
- `release/*` 是集成分支，用于部署，不能提交代码到此分支

所以，我们在实现分支管理的时候，可以根据上述不同分支类型，查看对应分支列表，进行分门归类。另外，根据分支名可以查询到迭代，进而可以查看该分支对应的需求、进度、负责人等。

同理，可以对 `mr` 列表按照当前迭代对应的分支名进行归类
- 每个 mr 的状态、提交人、审阅人、comments等
- 每个迭代对应的 mr 个数 

分类的更大的好处是，提 `mr` 的时候，不会提到其他分支，尤其是 `main`
- 之前大家都是在 `gitlab` 上提交 `mr`，但是它的默认目标分支是 `main`，很容易就直接提到 main 了
- 就算搜索到目标分支，那也要多一层搜索，比较费时间，尤其是分支数量很多的时候

### 部署相关
部署这块接的是阿里的云效，有2个要考虑的地方，一个要考虑是否开启分支模式，一个是配置流水线

```ts
interface BasementSchema {
  // ...
  devPipelineId: string // 开发环境流水线id
  betaPipelineId: string
  prodPipelineId: string
  branchMode: boolean
}
```

### 发布审批/上线日志

之前上线，都是要手动写个发布计划的文档，内容包括需求文档、开发人、MR、回滚方案等，效率低不说，压根起不到审批的作用，而且时间长了之后，发布计划文档，也是难以追溯。

现在接入平台之后，需求上线前会发起自动审批，前提是项目配置了「发布审批」，因为不是所有项目都需要审批的，比如技术项目组件库、三方库等。

```ts
interface BasementSchema {
  // ...
  needProdAudit: boolean
}
```

需求上线就会更新迭代状态为「已上线」状态，那么，查询「已上线」状态的迭代列表，再按日分类，上线日志就一目了然。

### appId

以上字段提交到服务器，会返回应用 `id` 即 `appId`。当在应用中创建新迭代，会把 `appId` 提交给服务端，这样应用和迭代就通过 `appId`关联起来

## 迭代

可以看到，应用层更多是添加相关配置，真正的干货都在迭代里。

### 名词解释
在此之前，先定义一下几个名词
- 需求：要实现的产品功能，一般通过 产品的需求文档 PRD 表述
- 迭代：一次完整的需求开发流程，包括需求文档、技术文档、UI稿、测试用例、相关人员、开发进度、日报、开发分支、排期等

### 元信息

根据定义，迭代对应的基本数据模型如下

```ts
interface IterationSchema {
  appId: string // 所属应用id，平台自动生成(创建应用返回的 appId)
  title: string // 本次迭代标题
  desc: string // 本次迭代描述

  prd: string // 需求文档
  ui: string // UI稿
  analysis: string // 系统分析,技术文档

  owner: IUser[] // 开发者
  pd: IUser[] // 产品人员

  status: ITER_STATUS // 进度
  dailyReportList: { // 日报列表
    content: string
    date: number
  }[]

  testDate: Number // 提测时间
  uatDate: Number // uat时间
  pubDate: Number // 上线时间
  
  branch: string // 分支名
  branchUrl: string // 分支url
  projectName: string // gitlab project name
  projectId: string // gitlab project id
}
```

其中 `ITER_STATUS`，表示迭代的进度，我们定义以下7种状态

```ts
enum ITER_STATUS {
  INIT = 1,
  DEV,
  TESTING,
  UAT,
  PUBLISHING,
  PUBLISHED,
  INVALID,
}
```

### 关联 Git 分支

一个迭代对应一个分支，在创建的时候，开发者需要填入分支名 `branch`，服务端然后调用 `gitlab/codeup api` 生成对应的远程分支。下面是使用 `gitlab api` 创建新分支的代码

```ts
const res = await axios.post(
  `${GITLAB_API_V4}/projects/${projectId}/repository/branches?branch=${branch}&ref=${ref}`,
  {},
  {
    headers: { 'Private-Token': GITLAB_PERSONAL_ACCESS_TOKEN },
  }
)

return {
  name: res.data?.name,
  web_url: res?.data?.web_url,
}
```

### 日报管理

以前，项目日报都只是发到项目群，并没有归档，不同需求想要汇总查看，还是比较麻烦的，要到处查看群消息。

现在，日报和迭代关联在一起，不仅支持一键发到群组，还可以查看历史日报，而且对所有人都是公开的。

要存储的信息就只有内容和日期，毕竟其他信息迭代本身就已经有了，而内容也不需要 `markdown`，寥寥几十字足以

```ts
dailyReportList: { // 日报列表
  content: string
  date: number
}[]
```

### 审批
以前是怎么上线的

- 编写发布计划，通知本周有发布权限的同学
- 对应同学手动合并代码到 `main`，然后再到云效生产环境发布
- 最后通知开发同学，发布完成，回归验证

整个流程非常被动，核心问题是真正负责该迭代的同学却没有发布的权限和对应的责任。

现在，当测试通过准备上线，开发者只需要提交审批流，审批者收到通知确认无误，同意发布即可，后续所有事项皆由开发同学操作。因此，每个迭代都要有至少一个审批者

```ts
interface IterationSchema {
  // ...
  audit: {
    userId: String,
    userName: String,
    agreed: Number,
    desc: String,
  },
}
```

审批通知，目前是接入到群组中，为此审批者还需要再次登录平台才能点击同意，稍微有点麻烦，不过也能用，更方便的是能接入 `IM` 的工作流。

### 导入迭代

业务中存在应用在多渠道发布的情况，比如同时发布到阿里、字节两个平台，这种情况大都使用 `monorepo` 架构。

这就给迭代平台造成2个小问题

1、在创建迭代的时候，一般都要判断 `branch` 是否已经创建，如果是就会提示 `分支已存在`

2、即使可以跳过分支存在检查，那同一个需求要在2个应用内反复创建，这就 `repeat yourself` 了


解决方法也很简单

- 第一个问题，跳过分支存在检查；
- 第二个问题，支持导入同一个 `project id`（注意，不是 `appId` 因为 `monorepo` 的项目 `project id` 相同但是 `appId` 不同） 下的迭代，导入其实等同于创建新迭代，只需要稍微修改部分迭代元信息，主要更新为当前应用的 `appId/appName`


## 发布
到目前为止，应用、迭代的 `CRUD` 层已经完成，只剩下最核心的部署流程未实现。部署提过很多次，接入的是 [阿里云效](https://help.aliyun.com/document_detail/472261.html)，要接入就要看下它提供了哪些 `API`

### Flow API

和部署息息相关的主要是这4个 `API`

- 运行流水线 `startPipelineRunWithOptions`
- 查询流水线运行详情 `getPipelineRunWithOptions`
- 查询每个阶段的日志 `logPipelineJobRunWithOptions`
- 取消流水线运行 `stopPipelineRunWithOptions`

整个服务只需要实例化一个 `flow` 对象即可，具体实现代码参考 [官方文档](https://next.api.aliyun.com/api/devops/2021-06-25/GetPipelineArtifactUrl?lang=TYPESCRIPT)

```ts
class FlowClient {
  client
  constructor(accessKeyId: string, accessKeySecret: string) {
    const config = new $OpenApi.Config({
      accessKeyId,
      accessKeySecret,
    })

    config.endpoint = `devops.cn-hangzhou.aliyuncs.com`

    this.client = new devops20210625(config)
  }
  static create(ack, ackSecret) {
    return new FlowClient(ack, ackSecret)
  }
  runPipeline(pipelineId: number, runningBranches: string[]) {}
  queryPipelineDetail(pipelineId: number, pipelineRunId: number) {}
  queryJobDetail(pipelineId: number, pipelineRunId: number, jobId: number) {}
  cancelPipeline(pipelineId: number, pipelineRunId: number){}
}
export default FlowClient.create()
```

### 状态机

一次发布至少有2个步骤：构建 和 部署。在 `flow` 平台，如果选择了分支模式，那么第一步是做分支集成，然后才是构建、部署。显然，整个流程是一个有限状态机


```ts
enum FLOW_STATE {
  BRANCH_CHECK = 'branch_check',
  BUILD = 'build',
  DEPLOY = 'deploy',
  DONE = 'done',
}

const stepList = [FLOW_STATE.BRANCH_CHECK, FLOW_STATE.BUILD, FLOW_STATE.DEPLOY, FLOW_STATE.DONE]

async function start() {
  let error
  for (; this.currentStep <= stepList.length - 1; this.currentStep++) {
    const step = stepList[this.currentStep]

    switch (step) {
      case FLOW_STATE.BRANCH_CHECK:
        error = await this.check()
        break
      case FLOW_STATE.BUILD:
        error = await this.build()
        break
      case FLOW_STATE.DEPLOY:
        error = await this.deploy()
        break
      default:
        break
    }

    if (error) {
      break
    }
  }
}
```

很多时候，我们希望在每一步切换的前后进行业务扩展，比如每一步完成之后都要进行某个操作等，第N步之前要做些前置判断等等，这就要求我们提供对应的 `hooks`，参考单元测试的语法，可以定义如下的钩子

- `beforeAll()`
- `beforeEach()`
- `before[FLOW_STATE]()`
- `afterAll()`
- `afterEach()`
- `after[FLOW_STATE]()`

```ts
async start() {
  for (; this.currentStep <= stepList.length - 1; this.currentStep++) {
    const step = stepList[this.currentStep]

    // 新增
    if (isFunction(this.options?.beforeEach)) {
      await this.options.beforeEach(this.currentStep, this)
    }
    const beforeCb = 'before' + to[0].toUpperCase() + to.slice(1)
    if (isFunction(this.options && this.options[beforeCb])) {
      await this.options[beforeCb](this.currentStep, this)
    }

    switch (step) {
      // ...
    }

    // 新增
    const afterCb = 'after' + to[0].toUpperCase() + to.slice(1)
    if (isFunction(this.options && this.options[afterCb])) {
      await this.options[afterCb](error, this)
    }
    if (isFunction(this.options?.afterEach)) {
      await this.options.afterEach(error, this)
    }

    if (error) {
      break
    }
  }
}
```

### Pipeline

状态机是一次发布的状态流转，而一次发布我们抽象为 `pipeline`。

首先是一次发布的状态，主要有4种情况

```ts
enum PIPELINE_STATE {
  WAITING = 'WAITING',
  RUNNING = 'RUNNING',
  SUCCESS = 'SUCCESS',
  FAIL = 'FAIL',
}
```

根据 `flow` 官方流水线操作，一次发布过程中涉及的主要操作包括

- 开始部署
- 查询流水线的详情
- 解决冲突
- 取消分支集成
- 取消部署

基于此，一个简单的 `pipeline` 类定义如下

```ts
class Pipeline {
  payload = {
    branch: '',
    projectId: '',
    appId: '',
    env: ENV_TYPE.DEV,
  }

  currentStep = 0
  state: PIPELINE_STATE = PIPELINE_STATE.WAITING

  // 状态机的开始、销毁
  start() {}
  destroy() {}

  private check() {}
  private build() {}
  private deploy() {}
  
  // 查询流水线的详情
  queryRunDetail() {}
  
  // 解决冲突
  resolveConflict() {}

  // 取消分支集成
  cancelRelease() {}

  // 取消部署
  cancel() {}
}
```

### 构建/部署

触发构建，其实就是调用 `flow api`

```ts
async function build() {
  const res = await flowIns.runPipeline(pipelineId, runningBranches)
  const pipelineRunId = res.pipelineRunId
}
```

问题是该接口返回的只是当下的流水线实例 `id` 即 `pipelineRunId`，如果想知道具体的流水线详情，还需要不断轮询查询详情的接口

```ts
class Pipeline {
  private async build() {
    const res = await flowIns.runPipeline(pipelineId, runningBranches)
    
    this.pipelineRunId = res.pipelineRunId
    
    this._startLoop() // 新增轮询
  }
  private async _startLoop() {
    this._runTimer = setInterval(async () => {
      await this.queryRunDetail()
    }, 10 * 1000)
  }
  async queryRunDetail() {
    const res = await flowIns.queryPipelineDetail(pipelineId, pipelineRunId)
  }
}
```

流水线详情接口返回结果如下，

```ts
{
  pipelineRun: {
    pipelineId: 123,
    pipelineRunId: 992,
    stages: [
      {
        name: 'Group0-Stage0',
        stageInfo: {
          jobs: [
            {
              id: 134762231,
              name: '构建',
              status: 'RUNNING',
            },
          ],
          name: 'Group0-Stage0',
          status: 'RUNNING',
        },
      },
      {
        name: 'Group1-Stage0',
        stageInfo: {
          jobs: [
            {
              id: 134762232,
              name: '主机部署',
              status: 'INIT',
            },
          ],
          name: 'Group1-Stage0',
          status: 'INIT',
        },
      },
    ],
    status: 'RUNNING',
  }
}
```
可以看到

- `pipelineRun.status` 表明了当前流水线的整体运行状态
- `pipelineRun.stages` 说明我们有2个步骤：构建、部署；每一步也有自己的 `status`

有了这些字段就可以更新状态机了

```ts
class Pipeline {
  private async build() {
    const res = await flowIns.runPipeline(pipelineId, runningBranches)
    
    this.pipelineRunId = res.pipelineRunId
    
    this._startLoop() 

    // 新增
    return new Promise((res) => {
      this._buildResolver = res
    })
  }
  // 新增
  private async deploy() {
    return new Promise((res) => {
      this._deployResolver = res
    })
  }
  async queryRunDetail() {
    const res = await flowIns.queryPipelineDetail(pipelineId, pipelineRunId)

    // 新增
    // 在运行
    if (res.pipelineRun.status == 'RUNNING') {
      const idx = res.pipelineRun.stages.findIndex((s) => s.status == 'RUNNING')

      if (idx - 1 == 0) {
        this._buildResolver && this._buildResolver()
      }
    }
    // 失败
    else if (res.pipelineRun.status == 'FAIL') {
      if (currentStep == 1) {
        this._buildResolver && this._buildResolver('构建失败，请查看日志')
      } 
      // 
      else if (currentStep == 2) {
        this._deployResolver && this._deployResolver('部署失败，请查看日志')
      }
    }
    // 部署成功
    else if (res?.pipelineRun?.status == 'SUCCESS') {
      this._deployResolver && this._deployResolver()
    }
  }
}
```

这段代码需要注意的点是，不同于以往的 `new Promise(handler)`，`resolve/reject` 会在 `handler` 中调用，这里我们存储了 `resolve` 的引用，只有当其他外界条件满足时，才会触发 `resolve()` 使 `promise` 变为 `fulfilled`。

```ts
// 普通模式
new Promise((resolve) => {
  setTimeout(() => resolve(), 1000)
}).then(() =>  console.log('haha'))

// 保存引用
let resolve
function waitMe() {
  return new Promise((res) => resolve = res)
}
waitMe().then(() => console.log('haha'))
setTimeout(() => resolve(), 1000)
```

### 分支集成

可惜的是，没有分支集成的 `API`。。。说起来当时我竟然没发现，上面的API都接入完了才发现无法分支集成，郁闷了1、2个星期才开始研究自己实现。

所谓分支集成，就是把不同分支按顺序 `merge` 到一个从 main 切的新分支上（约定为 `release/*`），然后部署该 `release` 分支。如果要自己实现，也不是很难

- clone project
- create a new branch named release/xxx if necessary
- merge each source branch into release/xxx
- commit and push release/xxx

代码实现如下

```ts
function createRelease(data: {
  sourceBranchList: {
    branch: string
    branchUrl: string
    commitId?: string
  }[]
  releaseBranch: string
  needCreateBranch: boolean
  env: number
  projectUrl: string
}) {
  const envDir = data.env == 1 ? 'dev' : data.env == 2 ? 'beta' : ''

  const baseDir = path.join(process.cwd(), envDir)
  await fs.ensureDir(baseDir)

  // clone project
  const git: SimpleGit = simpleGit({
    baseDir,
    binary: 'git',
  })
  const repoDir = path.join(baseDir, repoName)
  const exists = await fs.pathExists(repoDir)
  if (!exists) {
    await git.clone(data.projectUrl)
  }
  
  await git.cwd({ path: repoDir, root: true })

  // create a new branch named release/xxx if necessary
  let releaseBranch = data.releaseBranch
  try {
    if (data.needCreateBranch) {
      throw new Error('createNew')
    }
    else if (releaseBranch) {
      await git.checkout(releaseBranch)
      await git.pull()
    }
    //
    else {
      throw new Error('invalid releaseBranch')
    }
  } catch (e) {
    await git.checkout('main')
    await git.pull()

    releaseBranch = 'release/' + envDir + '-' + dayjs().format('YYYY-MM-DD-HH-mm-ss')
    await git.checkoutLocalBranch(releaseBranch)
    await git.push(['--set-upstream', 'origin', releaseBranch])
  }


  // merge branch
  for (const branch of data.sourceBranchList) {
    try {
      await git.merge(['--no-edit', '-s', 'recursive', '--no-ff', branch.commitId])
      await git.push()
    } catch (e) {
      await git.merge(['--abort'])
      throw new MergeException(e.message, { releaseBranch, commitId: branch.commitId })
    }
  }

  return {
    releaseBranch,
  }
}
function MergeException(message, detail?: any) {
  this.name = 'MergeException'
  this.message = message
  this.detail = detail
}
```

这段代码有2点要注意

1、不是每次部署都需要创建新的 `release/xxx`，比如在解决冲突的时候（具体见下面的解释）；那什么时候需要创建新的 `release/xxx` ？

- 有分支上线
- 有分支下线
- 有新分支加入

注意 `createRelease` 只负责保证有 release ，不关心到底需不需要创建新的 release，需不需要应该是上层业务决定，然后保存到 `this.lastRelease`


2、合并冲突

发生冲突是很常见的，问题是怎么通知给开发者让其手动解决，可以看到一旦发生冲突，代码就会抛出异常 `throw new MergeException`，所以我们要在第一步做分支集成的时候捕获冲突异常，再次使用 `保存promise引用` 的方法让状态机始终维持在第一步，直到开发者手动解决

```ts
class Pipeline {
  async check() {
    try {
      const data = await this.stepMerge()
      this.nextRelease = data
    } catch (e) {
      return e.message || 'check failed'
    }
  }

  private async stepMerge() {
    try {
      const data = await createRelease({
        ...this.lastRelease,

        env: this.payload.env,
        projectUrl: this.payload.projectUrl,
      })
      return data
    } catch (e) {
      if (/CONFLICTS/i.test(e.message)) {
        this.nextRelease.releaseBranch = e.detail?.releaseBranch

        // 注意这里，只有发生冲突，才会有 commitId，可以根据这个字段判断是否发生了冲突
        this.nextRelease.commitId = e.detail?.commitId

        if (isFunction(this.options.onConflict)) {
          this.options.onConflict(e.detail)
        }
      }
      return new Promise((res) => {
        // 注意这里，保存引用又出现了
        this._conflictResolver = res
      })
    }
  }
}
```

冲突一旦解决完成，需要开发者手动通知服务器，根据上面的代码，可知服务端会调用 `this._conflictResolver()`，推进状态机进入第2步【构建】。但这是不对的，因为分支集成还没结束，只是因为现在冲突发生了。怎么避免进入第2步呢？？很简单，状态机的运行顺序是通过 `currentStep` 控制的，强制修改回到第一步即可

```ts
class Pipeline {
  resolveConflict() {
    // 强制 -1，回到最开始，重新来过
    this.currentStep = -1
    this._conflictResolver(this.nextRelease)
  }
}
```


回到第一步之后，如果还是继续创建新的 `release`，那么依然还是有冲突，这样就进入了死循环。所以在 `stepMerge()` 还需要判断是否已经发生了冲突，如果是的，就使用旧的 `release`，这样才能保证旧的 `release` 已经知道该 `commit` 已经手动标记解决

```ts
class Pipeline {
  async stepMerge() {
    try {
      const data = await createRelease({
        // ...

        // 新增
        // 说明存在冲突，这时候不需要重新开新的集成分支
        ...(this.nextRelease?.commitId ? { needCreateBranch: false } : null),
        ...(this.nextRelease?.commitId ? { releaseBranch: this.nextRelease.releaseBranch } : null),
      })
      return data
    }
    // ...
  }
}
```

### 取消分支集成

最后如何取消分支集成？

1、要清空数据库中分支对应的 `release.commitId`，因为 `nextRelease.commitId` 表示冲突发生了，既然取消分支集成，这个标记就要清空避免发生未知错误；

2、要销毁本次部署的状态机

```ts
class Pipeline {
  async cancelReleaseDueConflict() {
    if (isFunction(this.options.onCancelRelease)) {
      const ok = await this.options.onCancelRelease({
        appId: this.payload.appId,
        env: this.payload.env,
        sourceBranchList: this.lastRelease?.sourceBranchList,
        releaseBranch: this.nextRelease.releaseBranch
      })
      if (ok) {
        this.error = '用户取消集成'
        this.state = PIPELINE_STATE.FAIL

        this.destroy()

        if (isFunction(this.options?.onCancel)) {
          await this.options.onCancel(this.error, this)
        }
        return ok
      }
    }
  }
}
```

### 取消部署

这里的部署，是指在 `flow` 平台存在一个运行中的流水线。

取消分支集成和取消部署的区别在于

- 分支集成是第一步，且是我们自己实现的，此时还没有到 `flow` 平台

- 取消部署，说明已经到 `flow` 平台了，此时查询流水线详情会存在一个 `pipelineRunId`，这时候取消，意味着取消 `flow` 平台的流水线

```ts
class Pipeline {
  // 取消部署
  async cancel() {
    const res = await FlowIns.cancelPipelineRun(this.pipelineId, this.pipelineRunId)
    if (res?.success) {
      this.error = '用户取消部署'
      this.state = PIPELINE_STATE.FAIL

      this.destroy()

      if (isFunction(this.options?.onCancel)) {
        await this.options.onCancel(this.error, this)
      }

      return res?.success
    }
  }
}
```

### 销毁发布

不管是取消分支集成、还是取消部署，最后都要做 `clean up` 当前 `pipeline` 的状态，避免内存泄露。我们的 `class pipeline` 存在轮询，所以需要及时销毁

```ts
class Pipeline {
  async destroy() {
    this._runTimer && clearInterval(this._runTimer)
    this._runTimer = 0
  }
}
```

### PipelineManager

`Pipeline class` 仅仅是单次部署的抽象，现实中，我们要实现多项目、多环境下同时部署，这就导致排队的问题，为了维护队列按顺序执行，我们要实现一个 `PipelineManager`

1、不同项目应该对应不同的 `manager`

```ts
class PipelineManager {
  static managerByProject = {}

  static create(options) {
    if (!PipelineManager.managerByProject[options.appId]) {
      PipelineManager.managerByProject[options.appId] = new PipelineManager()
    }

    const manager = PipelineManager.managerByProject[options.appId]
    manager.add(options)
  }
}
```

2、按环境区分同一个迭代同时间部署

3、当有 `pipeline` 在运行，其他 `pipeline` 加入队列

4、当运行完成，从队列中拿出最近的一条，开始运行

```ts
class PipelineManager {
  devPipelineList: Pipeline[] = []
  devRunning: Pipeline | undefined

  betaPipelineList: Pipeline[] = []
  betaRunning: Pipeline | undefined

  prodPipelineList: Pipeline[] = []
  prodRunning: Pipeline | undefined

  add(options: IOption) {
    const ins = new Pipeline({
      ...options,
      beforeEach() {},
      afterEach: (error: string, o: Pipeline) => {
        // 只要有一步发生错误
        if (error) {
          this.next(o.payload.env)
        }
      },
      afterDone: (error: string, o: Pipeline) => {
        // 都没出错，部署成功
        this.next(o.payload.env)
      },
    })
    if (data.env == ENV_TYPE.DEV) {
      // 加入队列
      this.devPipelineList.push(ins)
      // 尝试执行
      this.next(ENV_TYPE.DEV)
    }
    //
    else if (data.env == ENV_TYPE.TEST) {
      // 加入队列
      this.betaPipelineList.push(ins)
      // 尝试执行
      this.next(ENV_TYPE.TEST)
    }
    //
    else if (data.env == ENV_TYPE.PROD) {
      // 加入队列
      this.prodPipelineList.push(ins)
      // 尝试执行
      this.next(ENV_TYPE.PROD)
    }
    return ins
  }

  next(env) {
    const _next = (list, running: Pipeline) => {
      if (running && (running.state == PIPELINE_STATE.RUNNING || running.state == PIPELINE_STATE.WAITING)) {
        return running
      }
      
      if (!running) {
        const nextPipeline = list.shift()
        
        nextPipeline?.start()
        
        return nextPipeline
      }
    }

    if (env == ENV_TYPE.DEV) {
      this.devRunning = _next(this.devPipelineList, this.devRunning)
    } 
    else if (env == ENV_TYPE.TEST) {
      this.betaRunning = _next(this.betaPipelineList, this.betaRunning)
    } 
    else if (env == ENV_TYPE.PROD) {
      this.prodRunning = _next(this.prodPipelineList, this.prodRunning)
    }
  }
}
```


5、运行取消排队，从对列中删除

```ts
class PipelineManager {
  static cancelQueue(env, appId, id) {
    const manager = PipelineManager.managerByProject[appId]

    if (env == ENV_TYPE.DEV) {
      manager.devPipelineList = manager.devPipelineList.filter((p) => p.id != id)
    } 
    else if (env == ENV_TYPE.TEST) {
      manager.betaPipelineList = manager.betaPipelineList.filter((p) => p.id != id)
    } 
    else if (env == ENV_TYPE.PROD) {
      manager.prodPipelineList = manager.prodPipelineList.filter((p) => p.id != id)
    }
  }
}
```

## Summary

这个项目是所有基建中最复杂的，前后也做了有2个月，好在是真的提高了开发效率，大家用起来也很顺手。这篇文章写起来也非常费劲，拖拖拉拉的有2周，整体是按照我当时的开发思路来的，先做CRUD层，再研究如何部署，最后是分支集成。其实还有很多功能没有介绍，比如紧急发布、跳过分支检查、甚至是跳过整个发布（针对小程序项目）只保留上线记录等，这些其实也不是一开始我就能想到的，也是在大家不断使用中才发现，是该有这么个功能，慢慢补齐的。总之，这篇文章就到这吧。