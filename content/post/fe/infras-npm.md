---
title: "包发布管理平台"
date: "2023-12-11"
description: ""
tags: [基建]
categories : [
  "Frontend",
]
---

## Background

之前介绍过用 `verdaccio` 搭建私有 npm 源，公司内部用到的包可以通过 `npm publish` 发布上去。不过随着包越来越多，开发者也越来越多，只用 `npm publish` 已经不能满足现有的诉求了，主要问题在于


1、版本号可以手动修改，不一定严格遵守 `semver`，比较随意

2、缺少权限管控，谁都可以发布

3、没有发布记录，无法追溯

4、最大的问题是，**调试很麻烦**，有些人不会发带有 tag 的 debug 包，可能觉得自己代码没问题，先直接发到线上再说，那如果线上恰好发版的话就只能临时锁版本（这个情况遇到不下3次）

## What

一个规范的工作流应该如下

![](/blog/post/images/infras-npm.png)


Step 1 基于 main 新建 bugfix/feat，提交代码

Step 2 到发布平台，选择自己的分支，发布 debug 包

Step 3 更新业务的 pkg 版本号，进行测试

Step 4 测试OK，提交 MR；到发布平台，选择 main 发布正式包


除了 `npm publish` 有时候，我们也希望把包发到 `cdn`，所以图中多加了一个 `oss publish`，原理很简单，先构建得到产物，然后根据在平台指定的文件名，上传到 `oss` 即可


## How

### 版本规范

参考 [semver](https://semver.org/) && [npm-dist-tag](https://docs.npmjs.com/cli/v10/commands/npm-dist-tag)，不再赘述。


### 添加包

随着 `monorepo` 的流行，有些有依赖关系的包会放在一起方便管理，所以在平台录入包的时候，要考虑多包的可能。但无论是单包还是多包，包的元信息都是从 `package.json` 中获取。

此外，平台还需要设置权限管理，添加管理人员。

最后一个问题，不是所有的包都在 `pkg.json` 都指定了 `files` 字段。同样发布到 `cdn` 也是，不是都指定了 `jsdelivr` 字段。针对这种情况，平台支持手动配置

针对这些，包的数据结构设计如下：

```ts
interface IPackage {
  name: string
  gitlab: string
  maintainer: IUser[] // 维护者
  ossFilename?: sring // cdn 路径
  npmDirname?: string // npm 路径
  isMonorepo?: boolean
  packageList: IRepo[] // 子包列表
}
interface IRepo {
  name: string
  version: string
  ossFilename?: sring
  npmDirname?: string
}
interface IUser {
  userId: string
  name: string
}
```


#### 子包元信息

首先要知道项目的 `gitlab url/name`，然后下载到本地再解析 `packge.json` 从而得到 `name/version`

```ts
async findPackageByGitUrl(url, name, isMonorepo) {
  const tempDir = genTempDir()
  await gitClone(url, 'main', tempDir)
  
  const list = []
  await glob(isMonorepo ? 'packages/**/package.json' : '**/package.json', {
    ignore: ['node_modules/**', '**/node_modules/**],
    cwd: tempDir,
  }).then((paths) => {
    const pkg = fs.readJsonSync(path.join(rootDir, pathStr))
    
    if (pkg.private) return
    
    if (isMonorepo || pkg.name == name) {
      list.push({
        name: pkg.name,
        version: pkg.version,
      })
    }
  })

  return list
}
```

这段代码有个点要注意，基于 `monorepo` 的包不一定所有子包都是要发布的

- 比如 `utils` 可能是所有子包的公共依赖，但是不对外暴露，`package.json` 中大概率会配置 `private: true`，所以需要判断 `pkg.private == true`


- 比如开源的 `ui` 库 [element plus - github](https://github.com/element-plus/element-plus/tree/2.3/packages)，源码 `components/themes/utils/directives` 等采用 `monorepo` 组织（这些模块有自己的 `package.json`），但是打包的时候这些子包会完全平移到 `dist`，再通过 `dist/index.ts` 导入暴露出来。这种情况，在匹配 `package.json` 的时候，会拿到很多子包，但实际上只有一个是真正的包，所以需要手动指定真正的包名，并判断 `pkg.name == name`



#### 自定义配置

正如前文所说，平台需要支持配置 `npm/cdn` 路径等，具体如下

```ts
async addPackage(ctx) {
  const { name, gitlab, maintainer, npmDirname, ossFilename, isMonorepo, subRepoList } = ctx.request?.body || {}
  // 子包元信息
  const packageList = await findPackageByGitUrl(gitlab, name, isMonorepo)
  // 自定义子包配置
  if (isMonorepo && subRepoList?.length) {
    packageList = packageList.forEach((pkg) => {
    const curPackage = subRepoList.find((a) => a.name === pkg.name)
    if (curPackage) {
      pkg.npmDirname = curPackage.npmDirname || ''
      pkg.ossFilename = curPackage.ossFilename || ''
    }
  }
}
```


### 发布规范

#### beta版本约定

正如前文所述

- 非 `main` 分支发布，对应发布 `beta` 测试包

- `main` 则发布 `latest` 正式包


根据前面介绍的版本规范，约定 `beta` 测试包的版本命名规则为 `x.y.z-beta.[timestamp]`

```ts
function calcVersion(version, bumpType = 'patch', tag = 'beta' ) {
  const { major, minor, patch } = semver.parse(version)
  let nextVersion = semver.inc(`${major}.${minor}.${patch}`, bumpType)

  if (tag === TAG.BETA) {
    nextVersion += `-beta.${Date.now()}`
  }
  
  return nextVersion
}
```

#### 构建约定

如果是单包，那很简单，进入到对应目录执行 `pnpm run build` 就好


现在有多包的情况，为保持统一，约定构建命令如下 

```ts
pnpm run build:[subRepo]
```

其中 `subRepo` 即子包 `package.json`中 `name`字段值 `@scope/subRepo` 中的后半部分，用代码表示即

```ts
const subRepo = subModule.replace(/@\w+\//, '')}`
```

举个例子，多包 `elem-ui` 有如下子包


```ts
- directives: @elem-ui/directives
- utils: @elem-ui/utils
```

在根目录的 `package.json` 需要配置


```ts
{
scripts: {
 "build:directives": "pnpm -F @elem-ui/directives build",
 "build:utils": "pnpm -F @elem-ui/utils build",
}
```

### npm 权限

此外，平台还需要2个权限

- 向私有 `npm` 源的发布权限 => 可以发包

- `gitlab` 的读写权限 => 可以修改版本号


### 历史记录

最后，一切完成之后，把发布记录需要写入数据库。

```ts
await PublishRecordModel.create({
  package_name, // eg: elem-ui
  subRepo, // eg: @elem-ui/utils
  branch_name,
  tag, // latest/beta
  bump_type, // major/minor/patch
  version,
  status: 'success',
  publish_user: ctx.state.user?.name,
  publish_time: Date.now(),
  oss_url: ossUrl,
})
```

## Reference
- [semver](https://semver.org/)
- [npm-dist-tag](https://docs.npmjs.com/cli/v10/commands/npm-dist-tag)
