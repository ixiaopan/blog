---
title: "Common directives/plugins in Vue"
date: "2022-05-29"
description: ""
# tags: []
categories: [
  "Frontend",
]
---

总结一些常用的指令、插件

<!--more-->

## 防重复点击

```ts
/**
 * Prevent repeated clicks
 * @example v-throttle="500"
 */

import type { App, Directive, DirectiveBinding } from 'vue';
import { on } from './utils';

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

## 文本高亮

```ts
import type { App, Directive, DirectiveBinding } from 'vue'

function highlightMe(el, keyword) {
  if (!keyword) return
  
  const text = el.textContent
  const kwdRegExp = new RegExp(keyword, 'gi')
  el.innerHTML = text.replace(kwdRegExp, (match) => '<em class="highlight">' + match + '</em>')
}

const highlightDirective: Directive = {
  mounted(el: Element, binding: DirectiveBinding<string>) {
    highlightMe(el, binding.value)
  },

  updated(el: Element, binding: DirectiveBinding<string>) {
    highlightMe(el, binding.value)
  },
}

export function setupHighlightDirective(app: App) {
  app.directive('highlight', highlightDirective)
}
```

## 一键复制

```ts
/**
 * https://clipboardjs.com/
 * @example v-copy
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

```html
<div v-copy data-clipboard-text="you've copied me">
  copy me
</div>
```

## button-level access

### 需求

通常，业务需要支持按钮级别权限控制，一般情况控制显隐就足够了，但是我们这个有点麻烦，需要做到3点

- `disabled` 置灰
- `tooltip` 显示 『无权限』『不可下载』等提示文案
- 不可点击

直观的做法就是在各需要的地方，加上对应逻辑，比如

```ts
import { shouldDel } from 'xxx'

<a-tooltip :title="shouldDel() ? '删除' : '无权限'">
  <span :class="shouldDel() ? '' : 'disabled'" @click="onDel">删除</span>
</a-tooltip>

<a-tooltip :title="shouldDel() ? '删除' : '无权限'">
  <button :disabled="shouldDel()" @click="onDel">删除</button>
</a-tooltip>

function onDel() {
  if (!shouldDel()) return
}
```

这种写法有几个问题

- `button` 有 `disabled` 属性，可以直接屏蔽点击，但是普通标签 `div/span` 必须在事件中进行规避，缺少统一性
- `shouldXX()` 会被多次使用在 `html/js` 中，混乱且冗余
- 提示文案使用 `a-tooltip` 不仅 html 冗余，也和业务逻辑判断杂糅在一起

### 改进

这种需求用指令来封装再合适不过了，同样的功能但是只需要这样写就行

```html
<span v-access.del="permission">删除</span>
<button v-access.edit="permission">修改</button>
```

其中 `permission` 就是业务指定的权限，当在 `mounted` 时候，会判断是否需要 `disabled`

```ts
const accessDirective: Directive = {
  mounted: (el: Element, binding: DirectiveBinding<any>) => {
    checkAccess(el, binding)
    
    toggleDisableClass()
    disableClick()
    toggleTooltip()
  },

  updated: (el: Element, binding: DirectiveBinding<any>) => {
    checkAccess(el, binding)

    toggleDisableClass()
  },
}

function checkAccess(el, binding) {
  el.shouldDisable = shouldXX(binding.value)
}
```

指令 `v-access` 是怎么实现的呢？仔细分析一下这3个功能，其实最核心的是 **不可点击**，其他2个都可以用 `css` 解决

1、屏蔽点击事件`DOM` 本身就支持

```js
function disableClick() {
  el.addEventListener('click', () => {
    if (el.shouldDisable) {
      e.preventDefault()
      e.stopImmediatePropagation()
      e.stopPropagation()
    }
  }, true)
}
```

- `stopImmediatePropagation` 会阻止监听同一事件的其他 `event handler` 被调用
- `stopPropagation` 则阻止事件进一步向上冒泡
-  `e.preventDefault()` 阻止标签的默认行为


2、样式置灰

写一个全局的置灰样式 `.global-access-disabled`，根据权限动态修改 `class`

```ts
function toggleDisableClass() {
  if (el.shouldDisable) {
    el.classList.add('global-access-disabled')
  } else {
    el.classList.remove('global-access-disabled')
  }
}
```

3、提示文案

使用原生 `js` 实现，其实就是动态实现 `a-tooltip`，难点在于根据触发的元素设置 `tooltip` 的位置

```ts
function toggleTooltip() {
  const { showTooltip, hideTooltip } = createTooltip()
  el.addEventListener('mouseenter', () => { showTooltip(el.shouldDisable ? '无权限' : '删除') })
  el.addEventListener('mouseleave', hideTooltip)
}
```

如何计算 `tooltip` 位置？首先要知道当前元素的位置，可以使用 `Element.getBoundingClientRect()` 获取元素的 `top/left/width/height` 在根据业务需要是否居中、偏左、偏上等计算

```ts
function createTooltip() {
  if (!toolTipElem) {
    toolTipElem = document.createElement('span')
    document.body.appendChild(toolTipElem)
    toolTipElem.style.position = 'absolute'
  }

  function showTooltip(t: string) {
    if (toolTipElem) {
      toolTipElem.textContent = t
      toolTipElem.style.display = 'block'
      
      // 计算位置
      setPosition()
    }
  }
  function hideTooltip() {
    toolTipElem.style.display = 'none'
  }
  function setPosition() {
    const { left, top, width, height } = el.getBoundingClientRect()
    
    let tipTop = 0, tipLeft = 0
    if (placement == 'top') { // 上方居中
      tipTop = top
      tipLeft = left + width / 2
    }
    toolTipElem!.style.top = tipTop + 'px'
    toolTipElem!.style.left = tipLeft + 'px'
    toolTipElem!.classList.add(`global-access-tooltip-${placement}`)
  }

  return {
    showTooltip,
    hideTooltip,
  }
}
```

## 懒加载

按需加载不仅仅是指图片，任何资源如js、css都可以懒加载。比如只有1、2个页面才有上传的功能，那么可以按需加载对应的依赖库(比如ali-oss)。

### 图片

使用最新的 `intersectionObserver API`，如果不支持，全部加载不考虑降级(业务不需要，面向 `chrome` 开发)

```ts
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

```ts
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
