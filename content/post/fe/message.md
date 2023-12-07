---
title: "A message component"
date: "2022-06-03"
description: ""
# tags: []
categories: [
  "Frontend",
]
---

## What

`message` 不同于一般的组件，其生命周期非常短暂，只有在需要的时候，才会出现，然后快速被销毁，这就要求我们要动态的创建一条 `message`。不管是 `jQuery` 还是 `vue/react`，调用方法都是类似的

```ts
// jQuery
$.toast({
  text: 'hello world'
  duration: 5000
})
$.loading({
  text: 'loading...'
})

// vue
const msg = inject('message')
msg.info({
  message: 'hello world'
  duration: 5000
})
msg.loading({
  message: 'loading...'
  duration: 5000
})
```

通过 `jQuery` 我们知道，调用 `$.toast()` 实际上是动态的创建了一个含有 `text` 的 `html tag` 然后插入到 `body` 最后，


```ts
const div = document.createElement('div')
div.textContext = text

document.body.appendChild(div)

const timer = setTimeout(() => {
  document.body.removeChild(div)
  clearTimeout(timer)
}, duration)
```

类似的，`vue` 的实现也是如此，只不过我们是通过 `VNode` 生成真实的 `DOM`。


## Implementation

### render

```vue
<template>
  <div>
    {{ message }}
  </div>
</template>
```

以上定义 HTML 模板，接下来 '编译' 为 `VNode`，渲染为 `HTML` 插入到 `body`

```ts
import { createVNode, render } from 'vue'

import MessageConstructor from './message.vue'

const message = (options) => {
  const props = {
    zIndex: options.zIndex,
    offset: verticalOffset,
    ...options,
  }

  // create a VNode
  const vm = createVNode(
    MessageConstructor,
    props,
    null,
  )

  const container = document.createElement('div')
  
  // remove it from the document
  vm.props.onDestroy = () => {
    render(null, container)
    container = null
  }
  
  // transition starts
  vm.props.onBeforeLeave = () => {}

  // generate a html fragment
  render(vm, container)
  
  // insert into the document
  document.body.appendChild(container.firstElementChild)
}
```


### timer

`vue` 中我们在 `onMounted` 开启计时器，由于使用了 `transition` 实现动画效果，所以在动画结束即 `after-leave` 触发 `onDestroy` 移除 `DOM`


```vue
<template>
<transition @before-leave="onBeforeLeave" @after-leave="onDestroy">
  <div v-show="visible">
    {{ message }}
  </div>
</transition>
</template>

<script lang="ts">
function startTimer() {
  if (props.duration > 0) {
    timerId = setTimeout(() => {
      if (visible.value) {
        // trigger transition
        visible.value = false
        clearTimeout(timerId)
      }
    }, props.duration)
  }
}

onMounted(() => {
  startTimer()
})
</script>
```

### Multiple Instances

现在这段代码只支持一个 `message` 的创建，如果需要同时显示多个 `message`，这就要全局维护一个队列

- 创建的时候，添加到数组

- 移除的时候，从数组中移除

有一个 UI 要注意的细节，当移除第 `index` 个 `message`，它后面的 `message` 的 `offset` 要更新。一般 `message` 都是出现在上方，也就是说，后面的 `message` 的 `top` 都要减少，向上偏移。

这个操作可以放在 `before-leave` 完成，这样向上移动的效果和 `transition` 的效果就是同时的，体验上会更连贯。