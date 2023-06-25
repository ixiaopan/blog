---
title: "WDID - Frontend Project Structure"
date: "2023-06-07"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---


本文主要介绍一下当下 `general` & `vue-based` 前端项目结构划分


<!--more-->

## Naive 

The simplest project structure is like this,

```
test-demo
 - index.html
 - index.js
 - index.css
```

这种形式一般有两种情况： `demo` or `artifacts`。

复杂的项目当然不能像这样扁平的堆叠文件，可读性太差，难以维护。如果让我做一个初始化项目的模板，我会从以下几个模块考虑

## Project Metadata

对于一个从未见过的项目，熟悉它最快的方式莫过于浏览一下 `README.md/package.json`

### README.md

通常，我会写上项目的名字、描述、开发构建命令。这样，其他人看了 `readme.md` 就可以立刻启动项目

```md
# 周报定时提醒

## Install
npm i

## Dev
npm run dev

## Build
npm run build
```

### package.json

相较于 `readme.md` 像是一本使用手册，`package.json` 则是对项目元信息的描述。通常，我会更关注

- `dependencies`
- `scripts`

PS：如果是 `npm module`，我们还得关注 `version/exports/main/type` 等字段


```json
{
  "name": "report",
  "version": "0.0.1",
  "description": "周报定时提醒",
  "scripts": {
    "dev": "vite",
    "build": "vite build"
  },
  "dependencies": {
    "vue": "^3.2.20"
  },
  "devDependencies": {}
}
```


此时我们的项目结构如下

```
- test-demo
 - README.md
 - package.json
```

## Dev/Build

接下来，我们要对项目的开发、构建进行技术选型


### npm/yarn/pnpm

目前主流的3个 `package manager`，各有优劣，看自己的喜好和项目情况选择即可


### vite

这里我们使用 `vue3`，所以选择配套的 `vite` 进行构建。

不同的项目有不同的构建方式，对应的工具也是不同，比如
- `react` -> `webpack`
- `node module` -> `esbuild/rollup`


此时我们的项目结构如下

```
- test-demo
 - README.md
 - package.json
 - vite.config.ts
```

## Formatter

顾名思义，通过配置一定的规则，实现统一的代码风格和规范，团队合作开发必备。一般需要考虑

- 代码规范
  - `eslint/prettier`

- 提交规范
  - `commitlint`
  -  gitlab 配置 `hooks` 等强制校验

- 文件命名
  - 可开发 `vite/webpack` 插件 强制检查

- 分支命名
  - gitlab 配置 `protected branch / hooks` 等强制校验

### ignore

最经典的莫过于 `.gitignore` 了，那些不需要提交的文件都可以写在这里。常见配置如下

```bash
.DS_Store
node_modules
dist

# Log files
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
```

随着其他格式化工具的出现，对应的也会有自己的 `ignore` 文件，比如下文的 `.prettierignore/.eslintignore`

### eslint

### prettier

### commitlint

### editorconfig

这个文件用来约束编辑器的配置，比如针对不同文件类型 `js/md` 的格式化、空格、文件编码(兼容`OS`)等。常见配置如下

```bash
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
quote_type = single
```

此时我们的项目结构如下

```
- test-demo
 - .editorconfig
 - .eslintignore
 - .eslintrc
 - .gitignore
 - .prettierignore
 - .prettierrc
 - commitlint.config.ts
 - README.md
 - package.json
 - vite.config.ts
```

## Content

不管用不用框架开发，一些基本的开发模块都是相似的。

### pages

对于前端而言，最主要的就是入口 `index.html`，在各个框架(`react/vue`的加持下，现在我们一般去找 `views/pages` 等文件夹

很早之前，后端渲染的时候，入口的文件大都是 `.html` 结尾

```
pages
 - home.html
 - detail.html
 - about.html
```

现在SPA/CSR的时代，入口文件则和框架绑定，以 `vue` 为例

```
pages
 - home.vue
 - detail.vue
 - about.vue
```

但是不管什么 `suffix` ，本质还是 `html/js/css` 的集合，`page` 是一切的开始

### components

试想把所有内容都放在 `pages` 下的各个独立页面内，那

- 每个页面都会很大，至少是千行的代码量，可读性差
- 此外，相同的模块无法共享，存在重复开发，维护性差

`components` 就可以解决这些问题

记得第一次学习前端的时候，也是是创建了类似的文件夹 `partials` ，里面主要存放网站的公共模块，给后端渲染

```
partials
 - header.html
 - footer.html
 - nav.html
 - sidebar.html
 - carousel.html
```

如上，这些都是经典的功能模块，后端拿到这些模块，就可以自由组合成完整的页面了


### enums

枚举是我很喜欢的一个概念，它既是可变列表，值又有不可变的属性，同时自带表述能力。

试想面对如下的代码，如果没有注释，真的很难理解其意

```js
if (status == 50001) {

} else if (status == 50002) {

}
```
如果我们改写为 `enum` ，就大不相同

```js
enum STATUS_CODE {
  TOKEN_INVALID = 50001,
  REDIRECT_HOME = 50002
}
if (status == STATUS_CODE.TOKEN_INVALID) {

} else if (status == STATUS_CODE.REDIRECT_HOME) {

}
```

看似很简单，然而在工作中会发现，很多人从来不写 `enum`，导致项目中一堆不明其意的数字、字符串，只能跟着之前的逻辑继续「添砖加瓦」，导致项目越加难以维护


### utils

前端和浏览器交互是很复杂的，要涉及到浏览器的很多模块，比如 `dom/cache/http`，这些功能浏览器都有对应的 `API`，为了更好的维护和扩展，一般我都会每个 `API` 独立成文件，最后在 `index.ts` 全部导入

```
utils
 - dom.ts
 - util.ts
 - cache.ts
 - http.ts
 - index.ts
```


### assets/public

除了代码本身，前端还需要引入一些额外的资源，比如图片、字体、音视频等，一般我们会把他们放在 `assets` 

```
assets
- images
- fonts
- audio
```

有时候，我们也会看到 `public` 里面也存放一些图片，这两个区别在哪里


- `public` 是默认的静态资源公开路径，会原封不动的打包到 `dist` 下，然后部署到服务器下，所以一般建议

  - 资源不怎么更新的，比如空的兜底图；
  
  - 方便爬虫爬取的 `favicon/robots.txt`

- `assets` 会在打包构建的时候，根据不同的资源类型进一步处理，比如对于小于 `10Kb` 的图片，会被编译为 `base64` 减少请求，大于 `10KB` 的则在文件名加上 `hash`，防止缓存失效，一般建议
  
  - 图片比较小，可能会经常更新的
  
  - 组件自身需要一些图片比如 `icon` ，建议也打包进来，方便维护；不过如果图片体积过大，还是放在 `cdn` 比较好

### api/mock

最后就是和服务端的交互 —— 接口的请求，建议按模块划分，

- 各个功能模块比如 `login/order` 各自一个文件，避免耦合，

- 对于公共部分，比如全局的开关配置可以放在 `common` 里

```
api
 - login.ts
 - order.ts
 - common
   - index.ts
```

## Vue-Specific

### router

### store

### plugins

## Summary


最后的项目结构如下

```
- test-demo
 - mock
 - public
 - src
   - api
   - assets
   - components
      - header.html
      - footer.html
      - nav.html
      - sidebar.html
      - carousel.html
   - enums
      - cacheEnum.ts
      - httpEnum.ts
   - pages
      - home.vue
      - detail.vue
      - about.vue
   - plugins
   - router
   - store
   - utils
      - dom.ts
      - util.ts
      - cache.ts
      - http.ts
      - index.ts
 - test
 - .editorconfig
 - .eslintignore
 - .eslintrc
 - .gitignore
 - .prettierignore
 - .prettierrc
 - commitlint.config.ts
 - README.md
 - package.json
 - vite.config.ts
```



通篇看下来，看似我们在划分文件，实则我们在拆分功能，每个模块都只有自己唯一的功能，从而实现整个项目的高内聚，低耦合，这也是软件设计的一个原则

此外，通过项目结构的划分，我们也可以看到立项一个项目，需要考虑哪些因素

开发
  - 哪种技术框架 `vue/react/koa/nest/...`
  
  - 是否需要配套的 `store/router/middleware/...`

构建
  - `package.json` 配置哪些字段
  
  - `npm/yarn/pnpm` 怎么选择
  
  - `webpack/rollup/vite/esbuild/...` 用哪一个
  
  - 图片字体等静态资源放在哪里
 
规范
  - What 需要什么样的规范（代码、提交、文件、分支）？
  
  - How 怎么配置 `ts/eslint/stylelint/commitlint/prettier` 强制约束


然而很多人并不清楚项目结构划分的意义，大都是网上找个模板，能跑起来就行
