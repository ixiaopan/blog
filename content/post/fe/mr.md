---
title: "Automatically Send MR Notification"
date: "2023-01-28"
description: ""
# tags: []
categories: [
  "Frontend",
]
---

## Why


过去一年，团队开展了 MR 机制，但是效果不是很理想，原因有几个：

- 会 CR 的人太少
- 缺少时间
- 缺少 CR 规范
- MR 通知不及时
- ...

在人少需求不多的时候，这些问题都是可接受的。但是现在随着团队规模的扩大，这些问题就变得尖锐起来，这就需要规范且自动化，从而提高工作效率和代码质量。

回看上面的原因，其实也就2个大问题
- 不知道怎么CR
- 发起MR后，未及时感知

第一个问题的解决方法，可以是制定相关规范，让大家『有迹可循』，看多了也就知道问题在哪了；第二个问题，那就是自动化的问题，可以选择接入机器人，通知到 `IM`。


所以，这篇文章就是简单介绍一下如何接入 `gitlab` 的 `webhook` 实现 `MR` 的自动通知
## How

整体流程分为几步

- 拦截 MR 相关事件(open/update/close)

- 提取需要的信息

- 定时发送消息到 `IM`，这里我们用 飞书 举例


### Step 1 Webhook

`gitlab` 本身就提供了 `webhook` 允许自建应用响应 `gitlab` 相关事件，比如

- push
- tag
- issue
- merge request
- ...



其中 `Merge request` 又支持多个 `action`

- open
- close
- update
- merge
- ...


在 `gitlab` 的 `settings => Webhooks` 里，填写如下信息

- `webhook url` 即 `application endpoint`，比如 `https://examples.com/gitlab-mr`

- `security token` 会附加在请求头 `X-Gitlab-Token` ，进行安全校验，自定义字符串即可

- `Trigger` 根据自己的需求勾选，这里就是 `mr` 了




据此，我们定义以下 `enum` 

```js
// X-Gitlab-Token HTTP header
export const GITLAB_WEBHOOK_TOKEN = 'your custom string'

// 支持的事件类型
export enum GITLAB_EVENT_TYPE {
  // a new comment is made on commits, merge requests, issues, and code snippets
  COMMENT = 'note',
  // 
  MERGE_REQUEST = 'merge_request',
}

export enum MR_ACTION {
  open = 'open',
  reopen = 'reopen',
  update = 'update',

  merge = 'merge',
  close = 'close',

  approved = 'approved',
  unapproved = 'unapproved',
}

```

### Step 2 Server

搭建应用的话，可以选用 `koa` 起一个简单的 `node server` 

```js
import Koa from 'koa'
import KoaRouter from '@koa/router'
import { gitlabHookHandler } from './gitlab/handler'
import koaBody from 'koa-body'

const app = new Koa()
const router = new KoaRouter()

router
.get('/', () => {
  console.log('hello world')
})
// your own endpoint
.post('/gitlab-mr', gitlabHookHandler)

app.use(koaBody())
app.use(router.routes())

app.listen(3000)
```

#### API Validation

首先是边界处理，包括


- `Invalid Security Token` 这个就是前面 `settings => webhooks` 里的 `Security Token`

- `Empty Body`

- `Invalid EVENT_TYPE` 不是所有 `gitlab` 的事件都要处理

- `Invalid Projects` 只接受指定项目的 `webhook`


```js
export async function gitlabHookHandler(ctx) {
  // ...
  
  const { headers, body } = ctx.request
  
  // Token Validation
  const token = headers['X-Gitlab-Token'] || headers['x-gitlab-token']
  if (!token || token != GITLAB_WEBHOOK_TOKEN) {
    return responseError('invalid Secret Token')
  }

  // Empty Body
  if (!body) {
    return responseError('Empty Response')
  }

  const eventType = body.object_kind
  // Check Event Type
  if (!Object.values(GITLAB_EVENT_TYPE).includes(eventType)) {
    return responseError(`${eventType} event is not supported`)
  }
  
  // Project Validation
  const { id, name } = body.project || {}
  if (!id || !GITLAB_PROJECT_LIST.find(p => p.id == id)) {
    return responseError(`Project [${name}] does not exist`)
  }
}
```


一切都通过之后，就会进入 `MergeRequest Handler` 得到聚合后的数据，然后发送给 IM

```js
const data = await mergeRequestHook(body)
sendMR(data)
responseSuccess()
```

#### MR Handler

When do we need MR notifications? Here are some cases

- Open a MR with or w/o an assignee

- Close a MR

- Merge a MR


As mentioned before, `data.object_attributes.action` tells us which action is triggered.

```js
export async function mergeRequestHook(data: IMergeRequestEvent) {
  const { action, } = data.object_attributes
  
  // basic data about the project and MR
  const basicInfo = {
    url: '', // mr url
    title: '', // mr title
    description: '', // mr description
  }

  if (action == MR_ACTION.close) {
    return {
      ...basicInfo,
    }
  }
  if (action == MR_ACTION.open || action == MR_ACTION.reopen) {
    return {
      ...basicInfo,
    }
  }
  if (action == MR_ACTION.merge) {
    return {
      ...basicInfo,
    }
  }

  console.log(`MR action [${action}] is not supported`)
}
```

The above snippet does only one thing: collect data to be sent to IM.

#### UserId

For now, we have only data about the project and MR, but no user information. Specifically speaking, we don't have the user id in Feishu. There are 2 ways to tackle this

- save everyone's user id in the config file
- get userId by user's mobile/email via Feishu API (we need to apply for token)

Once we have the userId, we can send the above information to someone in Feishu. Here is an example,

```js
// the local user config file (Security Risk)
export const USER_LIST = [
  {
    id: 01, // userId in gitlab
    username: 'gitlab user name',
    userId: 'userId in Feishu',
  },
]

// 获取IM的用户id
export function getUserIdByGitId(id: string | number) {
  const m = USER_LIST.find(o => o.id == id)
  return m?.userId || ''
}
```

How to at someone?

```js
// set username as the default value, if id does not exist
"content": `<at id=${f.id}></at>${f.id ? '' : f.name}`
```

### Step 3 Schedule


以上实现的功能都比较被动，即需要等待事件被触发。假设想知道每天遗留的未处理MR的信息，then

- 主动发起 `API` 到 `gitlab`

- 设定每日定时提醒即可


首先申请 `Token`

```js
export const GITLAB_API_V4 = `${GITLAB_HOST}/api/v4/`

// TODO: Security Risk
export const GITLAB_PERSONAL_ACCESS_TOKEN = 'your token'
```

根据文档，只要知道 `projectId` 就能获取对应项目的 `MR` 列表，`Node` 端我们依然选用 `Axios` 发起 `HTTP` 请求，同时注意，要把之前申请的 `Token` 带到请求头

参考 
- [Gitlab Doc#personal-access-tokens](https://docs.gitlab.com/ee/api/rest/#personalprojectgroup-access-tokens)

- [GItlab Doc#list-project-mr](https://docs.gitlab.com/ee/api/merge_requests.html#list-project-merge-requests)


```js
const request = axios.create({
  baseURL: GITLAB_API_V4,
  timeout: 10 * 1000,
  headers: { 'Private-Token': GITLAB_PERSONAL_ACCESS_TOKEN }
})

export function fetchOpenedMRList(projectId: number) {
  return request.get(`/projects/${projectId}/merge_requests?state=opened`)
}

export async function createMRMention() {
  try {
    const allP = await Promise.all(GITLAB_PROJECT_LIST.map(async (p) => {
      const res = await fetchOpenedMRList(p.id)
      return res.data
    }))
    
    console.log('fetch opened mr done');
    
    allP?.forEach((data: IMergeRequest[]) => { // each project
      data.forEach((res: IMergeRequest) => { // each mr
        // ...
      })
    })

    sendRichText('MR 处理提醒', content)
  }
  catch (e) {
    console.error('fetchOpenedMR Error', e)
  }
}
```

最后，每天下午5点发起提醒

```js
const later = require('@breejs/later')

later.date.localTime()
const s = later.parse.text('at 5:00 pm')
later.setInterval(createMRMention, s)
```

## Summary
完整的代码，[点击这里](https://github.com/ixiaopan/DataScience/tree/master/GitlabWebhook)
