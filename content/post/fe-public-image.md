---
title: "Prefix static url when using public Image"
date: "2023-02-25"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---


<!--more-->

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

可以发现 `public` 用的是绝对路径。在打包的时候，会根据配置的 `baseUrl` 自动转换，比如配置的 `baseUrl = https://xxx-oss/static`，那么打包的产物就是

```html
<!-- // public -->
<img src="https://xxx-oss/static/images/xxx.png" />
```

## What I need

So far so good, but it's not what I need.

1、`public` 下图片每次打包都会复制到 `baseUrl` 所在的服务上

2、`public` 存储的几乎都是整个 app 共用的不会变的图片，比如空占位符、logo等

3、尽量让 `public` 简单，只存放 `favicon/robots.txt` 

出于这种目的，首先我把 `public` 下的图片都放在了 `oss` 上，然后在 `html` 中直接用 `http` 绝对路径引入 `<img src="https://xxx-oss/static/images/xxx.png" />`。但是时间一长，

- 代码里充斥着 `http` ，显得过于臃肿

- `url` 不好一键替换

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

综上，整个配置的目的就是，如果一张图片是不怎么变的，比如空占位符/兜底图，首先

1、上传图片到 `cdn`

2、在 `html` 中，这样引入 `<img src='./xx.png' />`

3、在 `css` 中，这样引入 `url('./xxx.png')`

4、如果动态，ze这样引入 `<DemoCom :src="$replaceStatic('xxx.png')" />`