---
title: "Extract the first frame of a gif image"
date: "2022-05-21"
description: ""
# tags: []
categories: [
  "frontend",
]
series: ["frontend"]
katex: true
---


不仅仅是 GIF，视频提取单帧画面也是一样的方法

<!--more-->


## Background

工作中遇到一个需求，GIF 默认显示第一帧画面，`hover` 的时候在显示 gif 本身，效果如下


![](/blog/post/images/gif-test.gif)



## How

`canvas.drawImage()` 可以提取序列帧，代码实现如下


```ts
function getFirstFrameFromGif(img) {
  const canvas = document.createElement('canvas')
  const ctx = canvas.getContext('2d')  

  canvas.width = img.naturalWidth
  canvas.height = img.naturalHeight
  
  try {
    ctx.drawImage(img, 0, 0, img.naturalWidth, img.naturalHeight)
    return canvas.toDataURL('image/png', 0.75)
  } catch (e) {
    return ''
  }
}
```


### Q1

拿到帧画面了，就需要交替显示图片。一开始是使用 `opacity/display` ，有几个缺点

- 显隐切换会抖动

- 底下的 `gif` 始终在播放，那么 `hover` 的时候，画面不是从头开始播放的

另个方法就是直接换 `src` 


```vue
<div class="gif" @mouseenter="toggleGif" @mouseleave="toggleGif">
  <img :class="['gif-placeholder', innerSrc == src ? 'gif-real' : 'gif-frame' ]" :src="innerSrc" />
</div>

function toggleGif() {
  if (!frameSuccess.value) return

  if (gifVisible.value) {
    innerSrc.value = frameSrc.value
  } else {
    innerSrc.value = props.src
  }
  gifVisible.value = !gifVisible.value
}
```

### Q2

这个组件接下来要显示在一个 `div` 里而且是 `cover` 整个容器的效果，但是组件用的 `img` 标签，无法直接用 `background-size: cover`，一个方法是用 `js` 手动算，但是还有更简单的方法，指定 `object-fit`

> The object-fit CSS property sets how the content of a replaced element, such as an `<img>` or `<video>`, should be resized to fit its container. 
> -- MDN


实现代码如下

```css
.gif {
  width: 100%;
  height: 100%;
}
.mogic-gif-frame {
  width: 100%;
  height: 100%;
  
  object-fit: cover;
}
```

