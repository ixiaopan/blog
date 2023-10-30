---
title: "D - Image Editor"
date: "2023-10-30"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---

业务中有不少对图片、视频甚至是PSD二次加工的需求，因此前端就需要提供对应的编辑器。从复杂程度来看，图片编辑器是最简单最容易实现的，涉及的常用功能主要是

- 显示（短边？长边）

- 放大缩小、滚动拖拽

- 平移旋转缩放

<!--more-->

## 图片显示

先看静态显示(不存在任何交互)，给定一个容器，显示一张图片，业务中会有几种情况

- 短边居中对齐，即要求图片保持缩放比同时最大程度包含在容器之内，也就是 css contain 效果

- 长边居中对齐，即要求图片保持缩放比同时最小程度超过在容器，也就是 css cover 效果

- 宽度撑满（保持宽高比）

- 高度撑满（保持宽高比）

PS：当然还有其他布局方式比如左上对齐、右下对齐等，不过不在业务中使用，就不在讨论范围内

主要有3种实现方式

### background

```html
<div class="cover"></div>
```

```css
.cover {
  background-size: cover; /* 长边 */
  background-size: contain; /* 短边 */
  background-size: 100% auto; /* 宽度对齐 */
  background-size: auto 100%; /* 高度对齐 */
}
```

- 简单高效，自动支持 `resize`

- 支持短边、长边、宽度、高度撑满


### object-fit


```html
<img src="https://xxx.png" />
```

```css
img {
  object-fit: cover; /* 长边 */
  object-fit: contain; /* 短边 */
}
```


- 简单高效，自动支持 `resize`

- 只支持短边、长边；无法满足宽度、高度撑满


当然 `object-fit` 还有其他取值，可以参考 [object-fit MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/object-fit)


### js

- 满足所有需求、灵活度高

- 但是需要适配 `resize`，考虑性能问题


首先定义4种剪裁方式

```ts
export enum CLIP_MODE {
  COVER = 1, // 长边 
  CONTAIN = 2, // 短边
  WIDTH = 3, // 宽度撑满
  HEIGHT = 4, // 高度撑满
}
```

对于长/短边，我们需要计算两个比例

- 容器的宽高比 `containerRatio = containerW / containerH`

- 图片的宽高比 `imageRatio = naturalWidth / naturalHeight`


通过比较这两个比例，计算图片缩放后的尺寸和位置


```js
const ratioX = containerW / naturalWidth
const ratioY = containerH / naturalHeight

const widthBase =
      mode == CLIP_MODE.COVER
        ? ratioX > ratioY
        : (mode == CLIP_MODE.CONTAIN ? ratioX < ratioY : mode == CLIP_MODE.WIDTH)

let imgWidth, imgHeight
if (widthBase) {
  imgWidth = containerW
  imgHeight = ratioX * naturalHeight 
} else {
  imgHeight = containerH
  imgWidth = ratioY * naturalWidth
}

const left = (containerW - imgWidth) / 2
const top = (containerH - imgHeight) / 2

setCSS(img, {
  left, 
  top,
  width: imgWidth,
  height: imgHeight,
})
```

从这段代码也能看出

- 短边的效果，一般是图片在容器内上下或者左右存在边距（也会有刚好的情况，容器和图片一样大）

- 长边则会溢出，被剪裁（left/top 是负值）


## 放大缩小+自定义滚动条

上文偏向静态显示，如果是 `cover` 效果，用户是无法查看被剪裁的地方的。有些时候，我们希望用户也能通过鼠标拖动、滚动等查看被剪裁的地方，这个就像是电脑上查看图片这个app的功能。


如果是这种情况，只能用 `js` 来实现，具体来说我们还需要实现如下功能

- 图片的平移

- 鼠标滚动监听

- 鼠标拖拽监听

- 自定义滚动条、滚动条拖拽


## 平移旋转缩放


## 架构设计

前面讨论的话题偏向某个具体的功能，现在我们从编辑器整体的角度来考虑，怎么整合上面的这些功能并能提供友好的API给业务方使用？

我设计一个类库，都会问自己一个问题

> 如果我是业务方，我要怎么方便的使用这个库？我肯定是希望一看到接口名，我就知道干啥的，下意识就知道传入那些参数，预期结果是什么



从这个目的出发设计的框架，我想是至少 API是不会不友好的，此外我还会考虑另外2点


- 扩展性好，比如这个图片编辑器，不仅支持图片、还支持视频、支持自定义对象等。
  - 但是我们不应该为了扩展而扩展，而是在开发之前要尽量能想到扩展性，如无必要，可以不扩展同时写好文档解释原因，或者给出另外的方法让用户自己实现

- 代码实现简洁且文档详实，这样方便业务方快速明白内部原理，实现自己的修改或扩展

