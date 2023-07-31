---
title: "D - Build your own UI component library from scratch"
date: "2023-06-07"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---


In this post, we will learn how to build your own UI component library based on monorepo.

<!--more-->


First of all, let's have a peek at the final project structure.

```
- ui-demo
  - docs
  - play
  - packages
    - components
      - button
        - __docs__
        - src
        - style
        - index.ts
    - themes
      - src
        - button.less
      - gulpfile.ts
      - package.json
    - utils
    - wxp-ui
      - index.ts
      - package.json
  - package.json
  - pnpm-workspace.yaml
  - README.md
```

## Initialize a project


First, we install the package manager `pnpm` globally. 

```js
npm i pnpm -g
```


Then we create an empty project and initialize it. In this tutorial, the project is called `ui-demo`. 

```ts
mkdir ui-demo

cd ui-demo
pnpm init -y
```

Next, install `vue & typescript` using `pnpm`, and then initialize the ts config file.

```ts
pnpm install vue@next typescript -D 

npx tsc --init
```

Generate the `.npmrc` and `pnpm-workspace.yaml` files under the root of `ui-demo`.

```js
touch .npmrc

# content
shamefully-hoist = true
```

Since we are using `vue + ts`, a declare ts file `vue-shim.d.ts` should be included.

```js
mkdir typings

cd typings

touch vue-shim.d.ts
```

Then copy the following content into `vue-shim.d.ts`.

```ts
declare module '*.vue' {
  import type { DefineComponent } from 'vue'
  // eslint-disable-next-line @typescript-eslint/ban-types
  const component: DefineComponent<{}, {}, any>
  export default component
}
```


Finally, declare the packages in pnpm config file `pnpm-workspace.yaml`.


```bash
touch pnpm-workspace.yaml

# the content
packages:
  - 'packages/**'
  - docs
  - play
  - '!**/__tests__/**'
```

Now, your project will look like this.

![](/blog/post/images/ui-demo-01.png)


## Playground

This is where you write components demos to debug components.

```bash
mkdir play
cd play

pnpm init -y
pnpm install vite @vitejs/plugin-vue -D
```

## Packages

在正式开发之前，有2件事要做

- 确定包名&命名规范

- 包模块划分


### 命名规范

确定包名及其命名规范，可以从以下几点考虑

- `package scope`
  
  - 包的私域，也就是 `@wxp/wxp-ui` 中的 `@wxp` 
  
  - 因为一般UI都是基于公司业务开发不会对外发布，而且公司内部有自己的包管理平台，对应的也有自己的私域，一般是公司名，这里我们以自己的名字作为私域 `@wxp`

- `package name`
  
  - 包的名字，即 `@wxp/wxp-ui` 中的 `wxp-ui` 

- `workspace name`

  - 因为这个组件库是基于 `monorepo` 的，他需要一个工作空间的概念，这里我们和 `package-name` 统一，约定为 `@wxp-ui`


- 组件命名
  - 可以选择 `upper-camel-case`、`kebab-style` 等


- 组件导出命名 `AButton`
  
  - 为了防止和其他引入的组件库冲突，比如 `antd` 的 `AButton` ，同时使自己的组件突出显示
  
  - 约定导出名字以 `W` 开头，比如 `import WButton from './button.vue'`


- 组件的类名、less 中的全局变量名
  -  参考 `antd` 的 `ant-`，我们也约定一个 `prefix` 比如这里的 `w-`


综上，我们确定命名规则如下

```
// package scope
scope: @wxp

// package name
name: wxp-ui

// pnpm workspace name
pnpm scope: @wxp-ui

// 组件文件名 kebab-style
input-number

// 组件导出名 驼峰式，以 W 开头
WButton
WIcon

// 组件的类名
.wxp-xx

// less 中的全局变量名，比如 .w-primary-color
abbr: w
```

### 包的划分

By default, packages are stored in the folder `packages`. Of course, you can change the name. In this project, we create four packages, as shown below.


```bash
mkdir packages
mkdir packages/components
mkdir packages/themes
mkdir packages/utils
mkdir packages/wxp-ui
```


Then, we initialize the first three package and add a scope for them by modifying the `name` field in the file `package.json`.


```bash
cd components && pnpm init -y
cd themes && pnpm init -y
cd utils && pnpm init -y
```

```
{
  - "name": "components",
  + "name": "@wxp-ui/components",
}
{
  - "name": "themes",
  + "name": "@wxp-ui/themes",
}
{
  - "name": "utils",
  + "name": "@wxp-ui/utils",
}
```

The last package is a bit special. In fact, it's the entry file that export all components.

```js
import * as components from '@wxp-ui/components'
import type { App } from 'vue'

export default {
  install: (app: App) => {
    Object.entries(components).forEach(([ name, comp ]) => {
      app.component(name, comp)
    })
  }
}

export * from '@wxp-ui/components'
```

The `package.json` for this package is served as the final `package.json` for the published package.

```json
{
  "name": "@wxp/wxp-ui",
  "version": "0.0.1-alpha.1",
  "description": "Vue UI Components",
  "exports": {
    ".": {
      "require": "./lib/index.js",
      "import": "./es/index.mjs"
    },
    "./es": "./es/index.mjs",
    "./lib": "./lib/index.js",
    "./*": "./*"
  },
  "main": "lib/index.js",
  "module": "es/index.mjs",
  "style": "dist/index.css",
  "unpkg": "dist/index.full.js",
  "peerDependencies": {
    "vue": "^3.2.20"
  },
  "repository": {
    "type": "git",
    "url": ""
  }
}

```

Now, install these packages at the `root` directory.


```bash
pnpm install @wxp-ui/components -w
pnpm install @wxp-ui/themes -w
pnpm install @wxp-ui/utils -w
pnpm install @wxp/wxp-ui -w
```

You can see that the above dependencies has been added into `ui-demo/package.json`.

```json
"dependencies": {
  "@wxp-ui/components": "workspace:^1.0.0",
  "@wxp-ui/themes": "workspace:^1.0.0",
  "@wxp-ui/utils": "workspace:^1.0.0",
  "@wxp/wxp-ui": "workspace:^0.0.1-alpha.1",
},
```


## Components

以 `Button` 为例，以下就是一个 `button` 组件需要的文件

```js
- packages/components
  - button
    - __docs__
      - button.md
    - __tests__ 
    - src
      - button.ts // props, types
      - button.vue // template + js
    - style
      - css.ts   // css module
      - index.ts  // less module
    - index.ts // entry
```

### index.ts

`index.ts` is the entry file. It does two things

- export the component 
- register the component globally


```ts
import { withInstall } from "@wxp/utils/with-install"
import Button from './src/button.vue'

// register
const WButton = withInstall(Button)

// export 
export {
  WButton
}
export default WButton
```


### src

```
src
- button.ts
- button.vue
```


`ts` 定义 组件的 `props`

```ts
import type { ExtractPropTypes } from "vue" 

export const buttonProps = {
  type: {
    type: String,
    default: 'primary'
  },

  size: {
    type: String,
    default: 'normal'
  },
}

export type ButtonProps = ExtractPropTypes<typeof buttonProps>

```

`vue` 就是普通的 `vue sfc` 


```vue
<template>
  <a-button :type="type" :size="size">
    <slot></slot>
  </a-button>
</template>

<script lang="ts">
import { Button } from 'ant-design-vue'

export default defineComponent({
  name: 'WButton',
  components: {
    [Button.name]: Button,
  },
});
</script>
```

### style

支持两种方式导入样式，`less & css`，可以看到

- 这个文件仅仅是导入样式

- 真正的样式是维护在 `theme/src/button.less`

- `theme/w-button.css` 则是编译后的产物


PS：这些文件无需手动编写，后续我们会有一个 `pnpm run create` 的命令，一键创建开发组件的模板


`css.ts`


```css
import '@wxp-ui/theme/base.css'
import '@wxp-ui/theme/w-button.css'
```

`index.ts`


```css
import '@wxp-ui/theme/src/base.less'
import '@wxp-ui/theme/src/button.less'
```

### doc

简单的方式，在每个组件内编写 `md`，然后在构建的时候，同步到 `packages/docs`，找个插件类似 `md2doc` 转为静态页面即可

```
button
  - __docs__
    - button.md
```

这种方式也有缺点，毕竟是面向开发同学的文档

- 缺少交互

- 无法给 UE 同学走查

为了兼容 开发/UE 同学，后面就升级为 `storybook` 了


## Theme

### 组件样式

按照约定，组件对应的样式名同组件名。可以看到，这里并没有 `theme/w-button.css` 只有源码 `button.less`

```
theme
 - src
   - button.less
   - index.less
 - gulpfile.ts
 - package.json
```

### 入口文件

除了组件样式之外，我们发现还多了一个 `index.less`，这是入口文件，会引入所有的组件样式，这是为了后续打包 `dist/inde.css` 做准备

```less
// component styles, add your component style from here
@import './button.less';
```

## Build

准备工作已经完成，现在就差打包构建了。我们希望支持

- js 支持 `cjs/esm` 引入

- js/css 支持全量导入 `dist/index.[js,css]` ，也可以支持按组件引入

换句话说，构建的产物大概是这个样子，才可以满足我们的需求

```
- dist
  - wxp-ui
    - dist
      - index.css // the whole css 
      - index.js // 
    - es // esm
      - components
        - button
          - src
          - style
          - index.mjs
      - index.mjs
    - lib // cjs
    - theme
      - w-button.css
      - index.css // the whole css 
      - src // the source code 
        - button.less
        - index.less 
    - package.json
    - README.md
```


针对这个目标，构建分为几个部分

- 打包js
- 打包css

### css

先看样式，这个最简单，就只有一个事情 `less2css`，所以首先安装 `less2css` 的相关插件

```sh
pnpm install gulp-less @types/gulp-less @types/sass gulp-autoprefixer @types/gulp-autoprefixer @types/gulp-clean-css gulp-clean-css -w -D
```

观察 `dist/wxp-ui/theme` 下的目录结构，可以看到

- 编译后的组件样式以 `w-` 开头

- `index.css` 保持名字不变

- `src` 是整个源码


```js
function compile() {
  return src(path.resolve(__dirname, './src/*.less'))
  .pipe(gulpLess())
  .pipe(
    rename((path) => {
      // `index.css` 保持名字不变
      if (!/index/.test(path.basename)) {
        path.basename = `w-${path.basename}`
      }
    })
  )
  .pipe(dest('./dist'))
}
// 复制编译后的产物 packages/theme/dist 到 `dist/wxp-ui/theme`
function copy2Theme() {
  return src(
      path.resolve(__dirname, './dist/**'))
      .pipe(dest(path.resolve(__dirname, distThemeDir)))
}

// 复制源码 packages/theme/src 到 `dist/wxp-ui/theme/src`
function copyThemeSource() {
  return src(path.resolve(__dirname, 'src/**')).pipe(
    dest(path.resolve(distThemeDir, 'src'))
  )
}
```

### js

- 一种简单的打包方式，是编译 `ui-demo/packages` 下所有的 `js/ts/vue` 结尾的文件

- 另外一种就是指定/过滤 `package` 或者再进一步结合 `js/vue/ts` 文件类型进行打包

不管哪种方式，我们的目的都是编译高级语法的`ts` 到兼容性更好的 `es6`语法的`cjs/esm`格式，

同时我还希望维持源码的目录层级，因为这个组件库后续我还会添加其他的包，比如 `plugins/utils` 等（他们可以选择作为子包单独发布，也可以集成在这个组件库里，通过 `import` 自动导入


#### Step 1 确定输入文件

```js
const input = excludeFiles(
  await glob('**/*.{js,ts,vue}', {
    cwd: packageRoot, // ui-demo/packages
    absolute: true,
    onlyFiles: true,
  })
)
export const excludeFiles = (files: string[]) => {
  // 进一步去除不相关的 `ts/js/vue` 
  const excludes = ['node_modules', 'test', '__tests__', 'mock', 'gulpfile', 'dist']
  return files.filter(
    (path) => !excludes.some((exclude) => path.includes(exclude))
  )
}
```

如果是方式2，我们可以调用 `pnpm/findWorkspacePackages` 遍历每个子包，明确的指定要编译的子包

```js
export const getDistPackages = async () =>
  (await findWorkspacePackages(projectRoot))
   .map(pkg => ({ name: pkg.manifest.name, dir: pkg.dir }))
   .filter(pkg=> !!pkg.name && !!pkg.dir 
    && !pkg.name.includes('theme') // 过滤掉 package/theme
  )
```

#### Step 2 build

打包可以使用 `esbuild/vue/commonjs` 等插件即可，但是有2个注意点

第一，我们发现 `button/style/index.ts` 中的引入路径不对，

```js
import '@wxp-ui/theme/src/base.less'
import '@wxp-ui/theme/src/button.less'
```

`@wxp-ui/theme` 是 `pnpm workspace name` 的域名空间，而我们要发布到 `npm` 的组件库的域名空间是 `@wxp`，换句话说，期望的引入路径是这样的

```js
import '@wxp/wxp-ui/theme/src/base.less'
import '@wxp/wxp-ui/theme/src/button.less'
```


那为什么不是 `import '../../theme/src/base.less` 呢？？因为我们希望保持外链导入，而不是在构建 `js` 的时候，再次打包 `css`，为此还需要告诉构建工具 导入模块路径或者 `id` 包含`theme` 的都是 `external` 这样就可以保持该文件内容不变（具体实现代码见下方）


为此，我们需要写个插件遍历每个文件

```js
export async function lessPathAlias() {
  return {
    name: 'less-path-alias-plugin',

    resolveId(id, importer, options) {
      if (!id.startsWith('@wxp-ui')) return

      const THEME_CHALK = `@wxp-ui/theme`
      if (id.startsWith(THEME_CHALK)) {
        return {
          id: id.replaceAll(THEME_CHALK, `@wxp/wxp-ui/theme`), // 这一行替换
          external: 'absolute', // 保持外部引入
        }
      }

      return this.resolve(id, importer, { skipSelf: true, ...options })
    },
  }
}
```

第2个问题是，组件依赖的第三库怎么处理，这里我偷懒了，选择让业务方主动安装，为此我在 `package/wxp-ui` 的 `package.json` 定义了 `peerDependencies` 

```json
{
  "name": "@wxp/wxp-ui",
  "peerDependencies": {
    "vue": "^3.2.20"
  }
}
```

```js
export const generateExternal = async () => {
  return (id: string) => {
    const packages: string[] = ['vue', 'wxp-ui/theme', '@vue', ...getPeerDependencies('package/wxp-ui/package.json')]

    return [...new Set(packages)].some(
      (pkg) => id === pkg || id.startsWith(`${pkg}/`)
    )
  }
}
```

最后用 `rollup` 打包即可

```js
const bundle = await rollup({
  input,
  plugins: [
    await lessPathAlias(),
    css(),
    vue({ target: 'browser' }),
    nodeResolve({
      extensions: ['.mjs', '.js', '.json', '.ts'],
    }),
    preprocessPlugin(),
    commonjs(),
    esbuild({
      sourceMap: true,
      target: 'es2018',
    }),
  ],
  external: await generateExternal(),
  treeshake: false,
})
```

## Debug

Usually, we use `npm link` to install our package globally, and then we install it in other projects. However, if you are using other versions of this package, it could cause confusion. Thus, I don't recommend this method.

The method I used is to write some bas scripts, like this,


```js
pnpm run build

cp -R dist/wxp-ui ../your-project/node_modules/@wxp/

cd ../your-project
rm -rf node_modules/.vite
```

Well, there are two things to note
- your project should have the level as the ui library
- you must restart your server if you use `vite`

Of course, you can improve this script further by watching file changes and restarting server automatically.


## Cli

最后为了方便一键创建组件模板，开发了一个简单的 `cli`，其实就是提前定义好模板，嵌入用户输入的组件名即可


```js
enum IAction  {
  ADD = 'add'
  CREATE = 'create'
}
type ITemplate = {
  type: IAction // 创建新文件、追加文件
  folder: string
  name: string
  content: string[]
}

function cli() {
  const file = path.resolve(compDir, t.folder, t.name)

  if (t.type == 'create') {
    // create folder only
    if (!t.name) {
      return fs.ensureDir(path.resolve(compDir, t.folder))
    }

    // create file
    return fs.ensureFile(file).then(() => fs.outputFile(file, t.content))
  }

  else if (t.type == 'add') {
    return fs.readFile(file, 'utf8').then(data => {
      let idx = 0
      const newContent = data.replace(/((?:\/\/|<!--)\s*<SLOT>)/g, (m, p1) => t.content[idx++] + '\n' + p1)
      return fs.outputFile(file, newContent)
    })
  }
}
```

## Summary

这一套下来，组件库是能用的，至少面向业务开发绰绰有余，但是离开源还差的很远，比如缺少测试、国际化、主题色等。开源组件库的设计逻辑肯定远比本文复杂，感兴趣可以更深入研究一下，就本文讨论的组件库设计就到此为止了。