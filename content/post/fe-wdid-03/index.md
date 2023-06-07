---
title: "WDID Build your own UI component library from scratch"
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

## Theme

按照约定，组件对应的样式名同组件名。可以看到，这里并没有 `theme/w-button.css` 只有源码 `button.less`

```
theme
 - src
   - button.less
 - gulpfile.ts
 - package.json
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


## Build

### Theme

```sh
pnpm install gulp-less @types/gulp-less @types/sass gulp-autoprefixer @types/gulp-autoprefixer @types/gulp-clean-css gulp-clean-css -w -D
```

### ESM


### Docs


### 产物

```
- dist
  - wxp-ui
    - dist
      - index.css // the whole css 
    - es
      - components
        - button
          - src
          - style
          - index.mjs
      - utils
      - index.mjs
    - lib
    - theme
      - w-button.css
      - index.css // the whole css 
      - src
        - button.less
        - index.less 
    - package.json
    - README.md
```

## Cli


## Usage

```sh
# create a new component
pnpm run create

# Build the ui components
pnpm run build

# Debug
pnpm run debug

# Playground
pnpm run dev

# publish
pnpm run pub
## after publishing, commit your content to the remote repo
git push

# Doc
## preview
pnpm run docs
## build
pnpm run docs:build

```
