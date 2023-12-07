---
title: "Naming conflicts in unplugin-vue-component"
date: "2023-02-25"
description: ""
# tags: []
categories: [
  "Frontend",
]
---

## Background


一般 `vue` 项目中都会用 `unplugin-vue-components/vite` 这个插件，用来动态引入 `vue` 组件，无需写 `import xx from xxx`。


```js
import Components from 'unplugin-vue-components/vite'
import { AntDesignVueResolver } from 'unplugin-vue-components/resolvers'

Components({
  // relative paths to the directory to search for components.
  dirs: ['src/components'],
  // search for subdirectories
  deep: true,
  // valid file extensions for components.
  extensions: ['vue'],
  // Allow subdirectories as namespace prefix for components.
  directoryAsNamespace: false,
  
  resolvers: [
    AntDesignVueResolver({
      importStyle: 'less',
    }),
    (name, a) => {
      if (name.startsWith('XX')) {
        const dirName = kebabCase(name.slice(1))
        return {
          importName: name,
        }
      }
    },
  ],
}),
```

但是它比较坑的是，项目中不能有同名文件。当然这个插件本身支持添加 `directoryAsNamespace: true` 解决这个问题，但是我个人倾向于不要有重名文件。

## Why naming conflicts

先看为什么为有重名问题？？下面就是这个插件的核心代码

```js
function getNameFromFilePath(filePath, options) {
 // Users/work/saas/src/components/XXXCenter/XXXCard/index.vue
  console.log('filePath', filePath);

  // 去掉前面的文件路径
  let strippedPath = "";
  for (const dir of resolvedDirs) {
    if (filePath.dir.startsWith(dir)) {
      strippedPath = parsedFilePath.dir.slice(dir.length);
      break;
    }
  }
  // /XXXCenter/XXXCard
  console.log('strippedPath', strippedPath);

  let folders = strippedPath.slice(1).split("/").filter(Boolean);
  let filename = parsedFilePath.name;
  // index, [XXXCenter, XXXCard]
  console.log('filename', filename, folders)

  // 如果组件是 index.vue 结尾的，使用所在目录名作为组件名
  if (filename === "index" && !directoryAsNamespace) {
    filename = `${folders.slice(-1)[0]}`;
    // 所以组件是 XXXCard
    console.log('real filename', filename)
    return filename;
  }
  
  // 如果目录作为空间，组件就是 XXXCenter-XXXCard
  if (directoryAsNamespace) {
    if (!isEmpty(folders)) {
      filename = [...folders, filename].filter(Boolean).join("-");
    }
    return filename;
  }
  return filename;
}

function updateComponent() {
  this._componentNameMap = {}
  
  const name = getNameFromFilePath(path)
  
  this._componentNameMap[name] = {
    as: name, 
    from: path
  }
}
```


如上，这个插件会默认遍历 `src/components` 的组件，从 `path` 中解析文件名作为组件名，然后缓存到 ` this._componentNameMap`。

后面在 `html` 遇到一个组件比如 `<CompName>` 就会先从 `_componentNameMap` 判断是否存在，不存在才会走到自定义的 `resolver`。

## Why resolver failed

走到自定义 `resolver`，这里又会遇到一个问题。如下代码，我们仅仅判断了 `name.startsWith('XX')` 就认为这个组件是合法的，但是有可能我们的组件库根本没有这个组件

```js
 (name, a) => {
  if (name.startsWith('XX')) {
    const dirName = kebabCase(name.slice(1))
    return {
      importName: name,
    }
  }
},
```

2个解决方法
- 自定义的组件库提供一个所有支持的组件名列表，多加一个判断
- 强制所有项目里的组件不要和组件库冲突，即不要以 `XX` 开头


## How

怎么解决项目内组件命名冲突？？方法有太多了，这里折腾了一个检测重名文件的插件，支持配置 目录、文件 `ext`，其实大部分代码都是借鉴 `unplugin-vue-components/vite`

一般而言，开发插件，就是在对应的钩子里做一些事情。对于我们的检测重名文件的插件，需要

- 初始化，配置目录、文件ext
- 初始化，检测项目是否有重名文件
- 监听文件变动，实时检测文件名

对应的 `vite` 插件的也就是2个钩子

- `configResolved`，这里可以拿到项目的 `root`
- `configureServer`，这里可以拿到 `vite` 的 `server` ，它自带一个 `file watcher`


```js

const defaultOptions = {
  dirs: ['src/components'],
  ext: 'vue',
}

type IOption = {
  dirs: string[],
  ext: string
}

export default function duplicateFileCheck(options: IOption) {
  const ctx = new Context()

  return {
    name: 'duplicate-file-check',

    configResolved(config: ResolvedConfig) {
      ctx.setupConfig({
        ...defaultOptions,
        ...options,
        root: config.root,
      })
    },

    configureServer(server: ViteDevServer) {
      ctx.setupServer(server)
    },
  }
}
```


剩下的就是初始化检查项目、实时监听文件变动，这个就属于插件自身逻辑了，这里就不在赘述了，完整的代码参考 [VitePluginDuplicateFileCheck - Github](https://github.com/ixiaopan/DataScience/tree/master/VitePluginDuplicateFileCheck)
