---
title: "开关配置"
date: "2023-08-07"
description: ""
tags: [hands-on]
categories: [
  "Frontend",
]
---

先说一下最简单的开关配置吧

<!--more-->

## Background

业务经常会有这种情况，对某个 `id` 先隐藏某个功能，待时机成熟在放开，此外这个白名单还可能随时追加。一般后端都有配置表，前端很少有。做这个开关就是实现这样的功能，当然，作用远不止如此，比如开关还可以当做数据配置平台，给运营使用用来实时更新页面数据等。

## How

要实现的功能：基于用户的配置，生成一个基于 `JSON` 的文件 `url`，提供给业务方使用


### UI

- 新建一个项目
  - 输入 projectId、projectName, projectDesc

- 创建成功，新建一个开关配置
  - 输入 id、name
  - 选择 类型（boolean or text)
  - 输入内容

- 点击保存，生成一个 `json` 文件的 `url`，比如 `https://xx.oss-xx/project-config.json`

- 业务引入这个 `json` 解析内容，使用即可


### 数据

#### 项目

项目除了自身的必要字段(id/name/desc)之外，还保存着开关列表、配置表的 `url` 这2个重要信息


```ts
{
  id: String, // 项目ID
  name: String, // 项目名
  desc: String, // 项目描述
  config: ISwitch[] // 开关列表
  
  url: String, // json配置文件url，系统生成
}
```

需要注意 `url` 是平台自动生成的，不用手动配置，比如我们可以使用 `projectId` 作为配置文件的 `url`，e.g. `https://xx.oss-xx/config/${id}-config.json`


#### 控件

初期我们只考虑 `boolean` 和 `string` 类型

- `boolean`，业务根据 `true/false` 判断功能是否可用等

- `string`，可以配置文本、json、url等可以序列化的数据


基于此可以确定一个开关的数据结构

```ts
type ISwitch = {
  id: string // 唯一id
  name: string // 名称
  disbaled: boolean // 是否被禁用
  control: string, // 控件类型
  content: string // 开关的内容
}
```

举几个例子，

1、`boolean` 开关

```ts
{
  "id": "approved",
  "name": "允许访问",
  "control": "boolean",
  "content": "false"
}
```

2、`string` 开关

```ts
{
  "id": "videoList",
  "name": "视频列表",
  "control": "string",
  "content": "[{"url": "https:"},{"url": "https:"}]"
}
```

### 保存

可以前端组装好数据，在前端上传到 `oss`，也可以后端组装/上传。具体上传就是调用 `OSS API`，下面的代码使用的是后端流式上传

```ts
const result = = await client.putStream(
  `${DIR_ON_OSS}/${id}.json`,
  stringToStream(JSON.stringify({ id, config }))
)

console.log(result?.url)
```