---
title: "Button-level access control"
date: "2022-11-10"
description: ""
# tags: []
categories: [
  "Frontend",
]
---

## Why

最近，业务上需要支持按钮级别权限控制，一般情况控制显隐就足够了，但是我们这个有点麻烦，需要做到3点

- `disabled` 样式

- `tooltip` 显示 『无权限』

- 不可点击


一种直观的做法就是在各个需要的地方，加上对应逻辑，比如

```html
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

- `a-tooltip` `disabled` 是在 `HTML` 中 `shouldXX()` 判断的，耦合度高

综上，核心问题是 `shouldXX` 造成的各种冗余，最终导致可维护性、可读性、可扩展性差，时间一长就没人敢去改这个代码了。

## How

既如此，我们只要找到一个方式能同时做这3件事情就好了，不管是组件、指令、或是其他模式。在 `vue` 中，我们可以使用指令去实现，同样的功能但是只需要这样写就行

```html
<span v-access.del="permission">删除</span>
<button v-access.edit="permission">修改</button>
```

具体实现还是要看指令内部 `v-access`，仔细分析一下这3个功能，其实最核心的是 不可点击，其他2个都可以用 `css` 解决。想要无侵入的屏蔽点击事件，一个方法就是拦截事件，好在 DOM 本身就支持


```js
el.addEventListener('click', () => {
  if (el.shouldDisable) {
    e.stopImmediatePropagation()
    e.stopPropagation()
  }
}, true)
```

- `stopImmediatePropagation` 会阻止监听同一事件的其他事件监听器 `event handler` 被调用

- `stopPropagation` 则阻止事件进一步向上冒泡


其实这也可以用来实现规避防重复点击，只是再多加一个时间限制而已。

完整的指令实现如下，

```js
function bindEvents(el, binding) {
  if (!binding.value) {
    return
  }

  const { showTooltip, hideTooltip } = createTooltip()

  // 显示 Tooltip
  el.addEventListener('mouseenter', () => {
    showTooltip(el.shouldDisable ? '无权限' : '删除')
  }, true)
  el.addEventListener('mouseleave', hideTooltip, true)


  // 屏蔽点击
  el.addEventListener('click', () => {
    if (el.shouldDisable) {
      e.stopImmediatePropagation()
    }
  }, true)
}
function checkAccess(el, binding) {
  el.shouldDisable = shouldXX(binding.value)
}
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
    }
  }
  function hideTooltip() {
    toolTipElem.style.display = 'none'
  }

  return {
    showTooltip,
    hideTooltip,
  }
}
```

