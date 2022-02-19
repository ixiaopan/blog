---
title: "Build your own frontend UI components from scratch"
date: "2022-01-23"
description: ""
# tags: []
categories: [
  "frontend",
]
series: ["frontend"]
katex: true
---


In this post, we will learn how to build your own UI components library based on monorepo.

<!--more-->


First of all, let's have a peek at the ultimate project structure.

```
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

```bash
npm i pnpm -g
```


Then we create an empty project and initialize it. In this tutorial, the project is called `ui-demo`. 

```bash
mkdir ui-demo

cd ui-demo
pnpm init -y
```

Next, install vue + typescript using `pnpm` and initialize the ts config file.

```bash
pnpm install vue@next typescript -D 
npx tsc --init
```

Generate the `.npmrc` and `pnpm-workspace.yaml` files under the root of `ui-demo`.

```bash
touch .npmrc

# content
shamefully-hoist = true
```

Since we are using `vue + ts`, a declare ts file `vue-shim.d.ts` should be included.

```bash
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


By default, packages are stored in the folder `packages`. Of course, you can change the name. In this project, we create three packages, as shown below.


```bash
mkdir packages
mkdir packages/components
mkdir packages/themes
mkdir packages/utils
```


Then, we initialize each package and add a scope for them by modifying the `name` field in the file `package.json`. Say, the scope name is `@wxp`


```bash
cd components && pnpm init -y
cd themes && pnpm init -y
cd utils && pnpm init -y

components -> @wxp/components
themes -> @wxp/themes
utils -> @wxp/utils
```


Next, install these packages at the root directory.


```bash
pnpm install @wxp/components -w
pnpm install @wxp/themes -w
pnpm install @wxp/utils -w
```

You can see that the above dependencies has been added in `package.json`.

```json
"dependencies": {
  "@wxp/components": "workspace:^1.0.0",
  "@wxp/themes": "workspace:^1.0.0",
  "@wxp/utils": "workspace:^1.0.0",
},
```


## Components

```
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


```less
import '@wxp/theme/base.css'
import '@wxp/theme/m-button.css'
```

`index.ts`


```less
import '@wxpi/theme/src/base.less'
import '@wxp/theme/src/button.less'
```


## Build

### Docs


### Themes

```
pnpm install gulp-less @types/gulp-less @types/sass gulp-autoprefixer @types/gulp-autoprefixer @types/gulp-clean-css gulp-clean-css -w -D
```

### ES



## Usage

```bash
# create a new component
pnpm run create

# Build the ui components
pnpm run build

# Doc
## preview
pnpm run docs
## build
pnpm run docs:build


# Playground
pnpm run dev

# publish
pnpm run pub
## after publishing, commit your content to the remote repo
git push
```
