---
title: "Composition bug in Safari"
date: "2022-05-21"
description: ""
# tags: []
categories: [
  "Frontend",
]
---

中文输入法 composition event 在 chrome 和 safari 表现相反

<!--more-->

## What

在开发编辑输入组件的时候，如果输入的是中文，监听 `delete` 事件会删除输入框里已经写好的文案，期望效果是删除正在输入的拼音。

bug 复现如下

![](/blog/post/images/safari-bug.gif)


期望效果

![](/blog/post/images/expected-safari.gif)


## Why

composition 有几个事件

- compositionstart
- compositionupdate
- compositionend

具体示例 MDN 上给了一个 [demo](https://developer.mozilla.org/en-US/docs/Web/API/Element/compositionstart_event) 可以体验

让我们看下中文输入 `asd`，然后 delete 后，发生了什么

chrome 事件顺序如下

- delete
- onCompositionUpdate

- delete
- onCompositionUpdate

- delete
- onCompositionUpdate
- onCompositionEnd


safari 事件顺序如下

- onCompositionUpdate 
- delete

- onCompositionUpdate 
- delete

- onCompositionEnd
- delete


如果输入法正在 `composition`，我们是知道的，只需要监听 `compositionstart, compositionupdate` 打个标记即可，然后在 `delete` 拦截掉

```js
function onCompositionStart() {
  isOnComposition = true
}
function onCompositionUpdate() {
  isOnComposition = true
}
function onCompositionEnd() {
  isOnComposition = false
}

function mockDelete() {
  // 拦截，false 的时候，删除业务 tag
  if (!isOnComposition && isMultipleMode && tags.value?.length && !phone.value) {
    const lastTag = tags.value[tags.value?.length - 1]
    removeTag(lastTag?._id)
  }
}
```

目前看没啥问题，但是到最后一个字符 `safari` 就和 `chrome` 表现不一样了，

因为 `compositonend` 在前 `delete` 在后，这就导致 `safari` 在监听到 `delete` 的时候，`isOnComposition` 已经被置为 `false` 了，这就导致会同时删除中文拼音以及业务 `tag`


解决方法也简单，在 `compositionend` 继续打个标记

```js
function onCompositionEnd() {
  isOnComposition = false
  mockDelete.safariBug = true
}

function mockDelete() {
  // safari bug: compositionEnd => delete
  if (isSafari && mockDelete.safariBug) {
    return mockDelete.safariBug = false
  }  
}
```
