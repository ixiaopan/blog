---
title: "Prefix static url when using public Image"
date: "2023-02-25"
description: ""
# tags: []
categories: [
  "Frontend",
]
---

## Background

项目中使用的图片一般会放在2个目录下

- `public`
- `assets`

这两个的区别，主要有2点

1、打包的时候会不会被 `webpack/rollup` 等识别为待打包的资源

- `assets` 的图片会根据图片大小打包为 `base64` 或者 `xxxx-[hash].[ext]`，
- `public` 还是保持原样

2、引用方式不同

```html
<!-- // public -->
<img src="/images/xxx.png" />

<!-- // assets-->
<img src="@/assets/images/xxx.png" />
```
`public` 用的是绝对路径，发布的时候，会放在 `html` 同级目录内。比如域名是 `xxx.test.com`，`public` 下的图片地址可能是 `xxx.test.com/common/empty.png`。

对于不频繁变更的图片比较适合放在 `cdn`，刚开始偷懒我就放在了 `public` 下，随着业务迭代，越来越多图片放在这里，这就给服务器带来不小的压力，所以我开始切换到 `cdn`

## What I want
使用 `cdn` 有个麻烦的地方，就是要引入绝对路径，但是我希望是这样的使用

```html
<img src="./empty.png" />
```

而不是这样的使用

```html
<img src="https://xxx-oss/static/empty.png" />
```

换句话说，HTML 不要充斥着 `https://xxxx`，对于开发者来说，只需要关心图片叫什么名字就好了。


## 尝试

### vite base

鉴于 `base` 就是最终的 `js/css` 地址，同时也是 `cdn`地址，所以我就想要不要都用 `base`，但是

- `baseUrl` 不一定就是 图片最终的 `http url`；换句话说，一般 `js/css` 是 `baseUrl`, 图片我们可能会有自己的图床服务，资源地址不同于 `baseUrl` 

- 需要本地项目的 `public` 里也有对应的图片文件，因为 `baseUrl` 在 `dev` 开发时配置的是 `/` ，而不是远程 `cdn` 地址，这就造成图片冗余、管理混乱


第一，`public` 简单，只存放 `favicon/robots.txt`；第二，既然 `public` 的图片已经在 `cdn` ，为什么还要在本地冗余、构建的时候重复上传一遍呢？？


### HTML 写死绝对路径

还是试试常规思路

- 图片先上传到 `oss` 
- 在 `html` 中直接用 `http` 绝对路径引入 

```html
<img src="https://xxx-oss/static/images/xxx.png" />
```

这样 `public` 只有 `favicon/robots.txt` ，但是也有问题

- 代码里充斥着 `http` ，显得过于臃肿
- `url` 不好一键替换

### HTML 写相对路径

我希望的是 `html` 只需要写相对路径，在构建的时候，可以自动转换。当然不是通过 `baseUrl`，因为图片的 `url` 是单独的资源服务器，和 `baseUrl` 是不一样的

```html
<img src="./xxx.png" />
```

所以，这两天花了点时间研究一下 `vite`，看看是不是可以通过配置解决，最后验证确实可以。


## How

### HTML


`vite` 为了解析 `sfc` ，引入了 `vitejs/vite-plugin-vue`，其内部引入了 `compiler-sfc`，这个包有一个文件叫 [templateTransformAssetUrl.ts](https://github.com/vuejs/core/blob/main/packages/compiler-sfc/src/templateTransformAssetUrl.ts)，顾名思义就是转换 `html` 里遇到的 `url`，这里有一段这样的代码


```js
if (options.base && attr.value.content[0] === '.') {
  // explicit base - directly rewrite relative urls into absolute url
  // to avoid generating extra imports
  // Allow for full hostnames provided in options.base
  const base = parseUrl(options.base)
  const protocol = base.protocol || ''
  const host = base.host ? protocol + '//' + base.host : ''
  const basePath = base.path || '/'

  // when packaged in the browser, path will be using the posix-
  // only version provided by rollup-plugin-node-builtins.
  attr.value.content =
    host +
    (path.posix || path).join(basePath, url.path + (url.hash || ''))
  return
}
```

所以我们只要在 `vite` 中配置 `base` 以及在 `html` 中用 `.` 开头即可，源码 [vite-plugin-vue/src/template#L137](https://github.com/vitejs/vite-plugin-vue/blob/eef8929c95d8b5cce1385a1d5e60da56a8420c0b/packages/plugin-vue/src/template.ts#L137)


```js
plugins: [
  vue({
    template: {
      transformAssetUrls: {
        base: VITE_COMMON,
        // tags: { // the default tags
        //   video: ['src', 'poster'],
        //   source: ['src'],
        //   img: ['src'],
        //   image: ['xlink:href', 'href'],
        //   use: ['xlink:href', 'href'],
        // },
      },
    },
  }),
]
```

改写后的 `html`，现在我们可以安全的删除 `public/images/xxx.png` 了

```html
<!-- // public -->
<img src="./images/xxx.png" />
```


上面的代码还支持可以配置自己的标签、属性用来加载图片，比如业务中封装了 `Empty.vue` ，希望 `vue` 能自动识别 `src` 的值为 `asset`，可以配置如下

```js
plugins: [
  vue({
    template: {
      transformAssetUrls: {
        base: VITE_COMMON,
        tags: { // the default tags
          video: ['src', 'poster'],
          source: ['src'],
          img: ['src'],
          image: ['xlink:href', 'href'],
          use: ['xlink:href', 'href'],
          
          empty: ['src'], // your custom tags
        },
      },
    },
  }),
]
```


### Less


如果是在 `css` 中使用了 `public/images/xxx.png`，又该怎么处理？？同样，我们在源码 [compiler-sfc/src/stylePreprocessors](https://github.com/vuejs/core/blob/main/packages/compiler-sfc/src/stylePreprocessors.ts#L122) 找到了如下的代码，其实就是利用了 `additionalData`


```js
function getSource(
  source: string,
  filename: string,
  additionalData?: string | ((source: string, filename: string) => string)
) {
  if (!additionalData) return source
  if (isFunction(additionalData)) {
    return additionalData(source, filename)
  }
  return additionalData + source
}
```

那么，`vite` 里就可以配置如下

```js
css: {
  preprocessorOptions: {
    less: {
      additionalData(source: string) {
        source = source.replace(new RegExp(`\\/xxxx\\/`), VITE_COMMON)
        return source
      },
    },
  },
}
```

### Dynamic 

以上都是静态引入的情况，还有一种情况是动态的，比如 `src` 是从父组件传进来的，这就无法解析，那这种情况，我们写一个插件就好了

```js
import type { App } from 'vue'

const slash = (s: string) => (s.endsWith('/') ? s : s + '/')
const unSlash = (s: string) => (s.startsWith('/') ? s.slice(1) : s)

export default {
  install: (app: App, options: { prefixUrl: string }) => {
    const { prefixUrl } = options || {}

    app.config.globalProperties.$replaceStatic = (path: string) => {
      return /^https?:\/\//.test(path) ? path : `${slash(prefixUrl)}${unSlash(path)}`
    }
  },
}
```

在任何需要动态引入的地方

```html
<DemoCom :src="$replaceStatic('xxx.png')" />
```

## Summary

综上，为了避免使用绝对路径的引入图片，首先

1. 上传图片到 `cdn`

2. 在 `html` 中，这样引入 `<img src='./xx.png' />`

3. 在 `css` 中，这样引入 `url('./xxx.png')`

4. 如果动态，这样引入 `<DemoCom :src="$replaceStatic('xxx.png')" />`
