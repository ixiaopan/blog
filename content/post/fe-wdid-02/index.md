---
title: "WDID Frontend Project Structure"
date: "2023-06-06"
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

```js
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

```bash
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

```bash
- test-demo
 - README.md
 - package.json
 - vite.config.ts
```

## Formatter

### ignore

### eslint

### prettier

### commitlint

### editorconfig

此时我们的项目结构如下

```bash
- test-demo
 - .editorconfig
 - .eslintignore
 - .eslintrc
 - .gitignore
 - .prettierignore
 - .prettierrc
 - commitlint.config.js
 - README.md
 - package.json
 - vite.config.ts
```

## Content

### page

### components

### enums

### assets

### api

### utils

## Vue-Specific

### router

### store

### plugins

## Summary


## TODO

- `./fe-package-json/`=> 常见容易混淆字段的含义
- `./fe-npm/`=> 哪里不同，哪里相同？为什么会有 `yarn/pnpm`？解决了什么问题
- `./fe-build/` => `gulp/rollup/vite/webpack` 源码分析
