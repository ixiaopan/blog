---
title: "Text ellipsis"
date: "2022-11-05"
description: ""
categories: [
  "Frontend",
]
---

## Why

文本截断并显示省略号，在web上是非常容易实现的，直接使用CSS就可以，那这有什么可以说的呢？道理是这样不错，但是业务总是千奇百怪。比如我们的业务是
- 如果文本太长超过容器的高度，则需要显示省略，并且还要tooltip显示原本的文案；
- 没超过容器高度，则正常显示；

换句话说，这就要求动态显示；这样的话，就无法通过CSS实现了

## How

从使用方的角度考虑，他只需要传入这几个参数

- text 原始文案
- rows 最多显示几行
- maxHeight 不确定几行，但是明确最大高度

Step 1 是否应该截断


```js
const visualH = props.maxHeight ? props.maxHeight : getLineHeight(container) * props.rows
const containerH = getElemHeight(container)

if (containerH <= visualH) {
  shouldTruncate = false
} else {
  shouldTruncate = true 
  truncate(text)
}
```

这里建议
- 要么明确指定可见高度, `maxHeight`
- 要么明确指定容器的行高, `lineHeight`


Step 2 如何截断

可以用笨方法，一次加一个字的方法判断是否超过了可视高度；这个效率有点低了，比较快的方法是用 `binarySearch` (折半查找)


```js
function binarySearch(l, r, callback) {
  while (l < r) {
    const m = Math.floor((l + r) / 2)
    if (callback(l, r, m)) { // 太长
      r = m - 1
    } else { // 太短
      l = m + 1
    }
  }
}

function truncate(text: string) {
  let finalText = ''
  binarySearch(0, text.length, (l, r, m) => {
    const partial = text.slice(l, m + 1)

    container.textContent = finalText + partial  
    const tempH = getElemHeight(container)
    if (tempH > visualH) {
      return true
    } else {
      finalText += partial
    }
  })
  container.textContent = finalText
}
```
