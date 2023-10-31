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

- 限制平移缩放&自定义滚动条

- 无限制平移、旋转、缩放

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


## 限制平移缩放&自定义滚动条

上文偏向静态显示，如果是 `cover` 效果，用户是无法查看被剪裁的地方的。有些时候，我们希望用户也能通过鼠标拖动、滚动等查看被剪裁的地方，这个就像是电脑上查看图片这个app的功能。


如果是这种情况，只能用 `js` 来实现，具体来说我们还需要实现如下功能

- 图片的平移、缩放
- 鼠标拖拽、滚轮缩放、双指缩放
- 自定义滚动条、滚动条拖拽

### 平移

先看平移，上文我们已经知道了图片的初始位置 `left/right` 和大小 `width/height`。同时我们知道，平移过程中，仅仅是位置的变化，大小不会变化，所以我们可以定一个方法处理平移

```js
private _translate(dx: number, dy: number) {
  if (!this.elem) return
  this.elem.style.transform = `translate(${this.left + dx}px, ${this.top + dy}px)`
}
```


接下来，只需要给容器绑定鼠标拖拽监听事件

```js
private _bindDrag() {
  const { container } = this.options
  
  let leftButtonPressed = false
  let isDraggingStart = false, isDragging = false, lastPosX = 0, lastPosY = 0
  let diffX = 0, diffY = 0

  container.addEventListener('mousedown', (evt: any) => {
    leftButtonPressed = evt.button == 0

    if (leftButtonPressed && this._shouldGrab()) {
      evt.preventDefault()
      evt.stopPropagation()
      
      isDraggingStart = true
      lastPosX = evt.clientX
      lastPosY = evt.clientY
    }
  })
  container.addEventListener('mousemove', (e: any) => {
    if (isDraggingStart) {
      isDragging = true
      
      diffX = e.clientX - lastPosX
      diffY = e.clientY - lastPosY

      this._translate(diffX, diffY)
    }
  })
  
  const cancelDrag = (e: any) => {
    leftButtonPressed = false
    isDraggingStart = false

    if (isDragging) {
      isDragging = false
      this.moveTo(this.left + diffX, this.top + diffY)
    }
    
  }

  container.addEventListener('mouseup', cancelDrag)
  container.addEventListener('mouseleave', cancelDrag)
}
```

### 限制平移

虽然实现了平移，但是我们会发现，图片可以被移动到无限远，超过容器边界。实际上，我们要实现的仅仅是容器大小内的平移

- 如果图片在容器之内，无法拖拽

- 反制，可以移动，但是最大只能移动到图片的边界
  
  - `this.top` 取值范围是 `[containerHeight - imgHeight, 0]`
  
  - `this.left` 取值范围是  `[containerWidth - imgWidth, 0]`


为满足这两个目标，我们要在鼠标 `mousedown` 的时候，计算出2个信息

1、判断是否图片是否完全在容器视口之内

```js
private _shouldGrabY() {
  return this.height > this._containerHeight
}
private _shouldGrabX() {
  return this.width > this._containerWidth
}
```

2、图片可移动的最大距离

既然图片已经可以移动，那么向左最大移动距离就是 `this.left`，向上最大移动距离就是 `this.top`，那么向右，向下，就需要下面的公式计算一下


```js
get remainedBottom() {
  return this.height - this._containerHeight + this.top
}
get remainedRight() {
  return this.width - this._containerWidth + this.left
}
```


最后，我们要调整一下 `mousemove` 

```js
container.addEventListener('mousemove', (e: any) => {
  if (isDraggingStart) {
    isDragging = true
    
    diffX = 0
    diffY = 0
    
    if (this._shouldGrabY()) {
      diffY = e.clientY - lastPosY

      // 正值:拖动图片向下,top值减少，最多减小到0
      diffY = diffY > 0 ? Math.min(Math.abs(this.top), diffY) : -Math.min(this.remainedBottom, -diffY)
    }

    if (this._shouldGrabX()) {
      diffX = e.clientX - lastPosX

      // 正值:拖动图片向右,left值减少，最多减小到0
      diffX = diffX > 0 ? Math.min(Math.abs(this.left), diffX) : -Math.min(this.remainedRight, -diffX)
    }

    this._translate(diffX, diffY)
  }
})
```

如果你仔细观察，会发现前面的代码 `cancelDrag` 多了一个 `this.moveTo(left, top)`，看样子是不同于 `_translate()`，但其实本质是一样的

- `_translate(dx, dy)` 是移动了多少距离

- `moveTo(left, top)` 是移动到某个位置，内部调用了 `_translate` 且同样做了 **平移限制** 的判断


```js
moveTo(nextLeft: number, nextTop: number) {
  if (!this._shouldGrabY() && !this._shouldGrabX()) return
  
  // 限制平移
  if (this._shouldGrabY()) {
    nextTop = Math.min(0, Math.max(this._containerHeight - this.height, nextTop))
  }

  // 限制平移
  if (this._shouldGrabX()) {
    nextLeft = Math.min(0, Math.max(this._containerWidth - this.width, nextLeft))
  }

  // 看着计算复杂，实际内部会抵消，本质就是移动到 `nextLeft/nextTop`
  this._translate(-this.left + nextLeft, -this.top + nextTop)
}
```

### 缩放

缩放既影响大小，也影响位置，再往细了说，缩放还需要缩放中心点，比如是按中心点、左上角、右上角等。缩放只需要指定缩放级别即可

```js
zoom(scale) {
  const nextW = this._naturalWidth * scale
  const nextH = this._naturalHeight * scale
  
  setCSS(this.elem, {
    position: 'absolute',
    width: nextW,
    height: nextH,
  })

  // 重新计算 `left/top` 会带来图片抖动问题，因为没有指定中心点
  this.moveTo((this._containerWidth - nextW) / 2, (this._containerHeight - nextH) / 2)
}
```

目前的实现有个问题，就是每次缩放都会重新计算 `left/top` 从而带来图片抖动问题，原因是没有按图片的中心点缩放，这里还值得进一步优化。


### 鼠标滚轮缩放/平移


### 自定义滚动条



## 无限制平移/旋转/缩放

终于我们实现了一版平移缩放，但是这个版本有很多不足之处：平移不能超过容器边界，缩放也只能按中心点缩放，而且还不支持旋转。

而真正意义上的图片编辑，元素的位置是可以超出画布之外，支持的功能也不仅限于平移、缩放，同时会有控制点支持不同方向的缩放。



## 架构设计


前面讨论的话题偏向某个具体的功能，现在我们从编辑器整体的角度来考虑，怎么整合上面的这些功能并能提供友好的API给业务方使用？

我设计一个类库，都会问自己一个问题

> 如果我是业务方，我要怎么方便的使用这个库？我肯定是希望一看到接口名，我就知道干啥的，下意识就知道传入那些参数，预期结果是什么



从这个目的出发设计的框架，我想是至少 API是不会不友好的，此外我还会考虑另外2点


- 扩展性好，比如这个图片编辑器，不仅支持图片、还支持视频、支持自定义对象等。
  - 但是我们不应该为了扩展而扩展，而是在开发之前要尽量能想到扩展性，如无必要，可以不扩展同时写好文档解释原因，或者给出另外的方法让用户自己实现

- 代码实现简洁且文档详实，这样方便业务方快速明白内部原理，实现自己的修改或扩展
