---
title: "Build your own UI component library from scratch"
date: "2022-01-23"
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


First of all, let's have a peek at the finished project structure.

```js
- vue-ui
  - docs
  - play
  - packages
    - components
    - themes
    - utils
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


By default, packages are stored in the folder `packages`. Of course, you can change the name. In this project, we create four packages, as shown below.


```bash
mkdir packages
mkdir packages/components
mkdir packages/themes
mkdir packages/utils
mkdir packages/wxp-ui
```


Then, we initialize the first three package and add a scope for them by modifying the `name` field in the file `package.json`. Say, the scope name is `@wxp-ui`


```bash
cd components && pnpm init -y
cd themes && pnpm init -y
cd utils && pnpm init -y

components -> @wxp-ui/components
themes -> @wxp-ui/themes
utils -> @wxp-ui/utils
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

Now, install these packages at the root directory.


```bash
pnpm install @wxp-ui/components -w
pnpm install @wxp-ui/themes -w
pnpm install @wxp-ui/utils -w
pnpm install @wxp/wxp-ui -w
```

You can see that the above dependencies has been added in `package.json`.

```json
"dependencies": {
  "@wxp-ui/components": "workspace:^1.0.0",
  "@wxp-ui/themes": "workspace:^1.0.0",
  "@wxp-ui/utils": "workspace:^1.0.0",
  "@wxp/wxp-ui": "workspace:^0.0.1-alpha.1",
},
```


## Components

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

To avoid conflicting with the vanilla HTML element, our components should have a prefix name, say `M`. For example, the button component name is `MButton`.



### index.ts

`index.ts` is the entry file. It does two things

- export the component when you use it in your project

- register the component globally when using `app.use(vue-ui)`


```ts
import { withInstall } from "@wxp/utils/with-install"
import Button from './src/button.vue'

const MButton = withInstall(Button)

export {
  MButton
}
export default MButton
```


### src

```vue
<template>
  <a-button :type="type" :size="size">
    <slot></slot>
  </a-button>
</template>

<script lang="ts">
import { Button } from 'ant-design-vue'

export default defineComponent({
  name: 'MButton',
  components: {
    [Button.name]: Button,
  },
});
</script>
```

### style

There are two ways of importing css.


`css.ts`


```css
import '@wxp/theme/base.css'
import '@wxp/theme/m-button.css'
```

`index.ts`


```css
import '@wxpi/theme/src/base.less'
import '@wxp/theme/src/button.less'
```

## Debug

Generally, we can use `npm link` to install our package globally, and then we install it in other projects. However, if you are using other versions of this package, it could cause confusion. Thus, I don't recommend this method.

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

### Themes

```sh
pnpm install gulp-less @types/gulp-less @types/sass gulp-autoprefixer @types/gulp-autoprefixer @types/gulp-clean-css gulp-clean-css -w -D
```

### ESM


### Docs


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
