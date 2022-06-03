---
title: "Common directives/plugins in Vue"
date: "2022-05-29"
description: ""
# tags: []
categories: [
  "frontend",
]
series: ["frontend"]
katex: true
---

Directives and plugins are different from third-party libraries. Usually, they are used to do simple tasks, such as lazy load image, format text, and so on.

<!--more-->

## 注册指令/插件

注册指令
```vue
app.directive('name', directive)
```

注册插件
```vue
export default {
  install: (app: App, options) => {
    app.component('foo', fooComp)
    app.directive('lazy', lazyDirective)
  }
}
```

可以看到，插件其实更为灵活，你可以在插件里注册组件、注册指令等。举个例子，

如果需要做全局配置，插件再好不过了。比如之前做懒加载图片的时候，一开始是写成一个指令，后来希望支持 `webp` ，也就只需要在 `url` 后添加参数即可。当然可以直接写死在指令里，不过这也太死了，所以打算加开关，这就改成了插件模式，在插件里注册一个指令。这样，既支持全局开关统一开启 `webp`，也支持单个图片是否开启 `webp`。


```vue
let enableWebp

const lazyDirective: Directive = {
  mounted(el: HTMLElement, binding: DirectiveBinding<any>) {
    // 图片自己的开关
    el.webp = binding.value?.indexOf('webp') > -1
  },
}

export default {
  install: (app: App, options) => {
    // 全局开关
    enableWebp = options?.webp || false    
    app.directive('lazy', lazyDirective)
  }
}
```

## 防重复

```vue
/**
 * Prevent repeated clicks
 * @Example v-throttle="500"
 */

import type { App, Directive, DirectiveBinding } from 'vue';
import { on } from '@mogic-ui/utils';

const throttleDirective: Directive = {
  beforeMount(el: Element, binding: DirectiveBinding<any>) {
    let timer
    
    const threshold = binding.value || 1000

    on(el, 'click', (e) => {
      if ((e as any).button !== 0) return;

      if (!timer) {
        timer = setTimeout(() => {
          clearTimeout(timer)
          timer = null
        }, threshold);
      } else {
        e.stopImmediatePropagation()
      }
    }, true)
  },
}

export function setupRepeatDirective(app: App) {
  app.directive('throttle', throttleDirective)
}
```


Then We can use `v-throttle` like this

```vue
<button v-throttle>Submit</button>
```

## 一键复制

```vue
/**
 * https://clipboardjs.com/
 * @Example v-copy
 */

import type { App, Directive, DirectiveBinding } from 'vue'
import Clipboard from 'clipboard'

import { Message } from './components/message'
import { on } from './utils'

const copyDirective: Directive = {
  beforeMount(el: Element, binding: DirectiveBinding<any>) {
    on(el, 'click', () => {
      const clipboard = new Clipboard(el)

      clipboard.on('success', () => {
        Message.success('复制成功')
        clipboard.destroy()
      })
      
      clipboard.on('error', () => {
        Message.success('复制失败')
        clipboard.destroy()
      })
    }, true)
  },
}
 
export function setupCopyDirective(app: App) {
  app.directive('copy', copyDirective)
}
```

Then We can use `v-copy` like this

```vue
<div v-copy data-clipboard-text="you've copied me">
  copy me
</div>
```

## 懒加载

按需加载不仅仅是指图片，任何资源如js、css都可以懒加载。比如只有1、2个页面才有上传的功能，那么可以按需加载对应的依赖库(比如ali-oss)。

### 图片

使用最新的 `intersectionObserver API`，如果不支持，全部加载不考虑降级(业务不需要，面向 `chrome` 开发)

```vue
let enableWebp
let previewHost

class Lazyload {
  io: any

  constructor() {
    this.io = this.initObserve()
  }

  initObserve() {
    if (!intersectionObserverEnabled) {
      return
    }

    return new IntersectionObserver(entries => {
      entries.forEach(item => {
        const elem = item.target as HTMLElement

        if (item.isIntersecting) {
          this.load(elem)
            .then(() => {
              this.io.unobserve(elem)
            })
            .catch(() => {
              console.log('loading image error');
            })
        }
      })
    })
  }

  observe(elem: HTMLElement) {
    if (!elem.getAttribute('data-src')) return

    if (this.io) {
      this.io.observe(elem)
    } else {
      this.load(elem)
    }
  }

  load(elem: HTMLElement) {
    return new Promise<void>((resolve, reject) => {
      let src = elem.getAttribute('data-src')

      if (!src) return resolve()

      const isImageNode = elem.nodeName.toLowerCase() === 'img'

      const img = isImageNode ? elem : new Image()

      // 使用 webp 的话
      // @ts-ignore
      const webpUrl = enableWebp || elem.webp ? (webpSupported() ? src + webpExt : src) : src

      on(img, 'load', () => {
        if (!isImageNode) {
          // @ts-ignore
          elem.style['background-image'] = 'url(' + webpUrl + ')'
        }

        elem.removeAttribute('data-src')

        resolve()
      })

      on(img, 'error', () => {
        elem.removeAttribute('src')
        reject()
      })

      img.setAttribute('src', webpUrl)
    })
  }

  
}

let lazy
const lazyDirective: Directive = {
  beforeMount() {
    if (!lazy) {
      lazy = new Lazyload()
    }
  },

  mounted(el: HTMLElement, binding: DirectiveBinding<any>) {
    lazy.observe(el)
  },

  updated(el, binding: DirectiveBinding<any>) {
    lazy.observe(el)
  },

  beforeUnmount() {
    lazy.io?.disconnect()
  }
}
```

### js

正如之前提到上传功能，
- 只有用户使用了上传才加载额外的第三方库
- 上传本身就会 `loading`，所以延迟加载不会有什么体验问题

```vue
const customCache = new Set()
function loadScriptFromRemote() {  
  if (customCache.has(scriptUrl)) {
    return resolve()
  }
  
  const script = document.createElement('script')
  script.setAttribute('src', scriptUrl)
  
  script.onload = () => {
    resolve()
  }
  script.onerror = () => {
    reject()
  }

  customCache.add(scriptUrl)
  document.body.appendChild(script)
}

let url = ''
const remoteJSDirective: Directive = {
  beforeMount(el: Element, binding: DirectiveBinding<any>) {
    loadScriptFromRemote(url)
  },
}

export default {
  install: (app: App, options) => {
    // TODO: 可以扩展到 url collections
    if (!options?.url) {
      return console.error('url is required')
    }

    url = options?.url

    app.directive('v-remote', remoteJSDirective)
  }
}

```