---
title: "Design a video player"
date: "2023-06-21"
categories: [
  "Frontend",
]
---

业务最开始实现的视频播放功能用的是 `xgplayer` 插件，这个插件比较重，更侧重于直播。而我们业务所需要的仅仅是一个美化版本的 `<video>`，为此，我们需要基于 `video api` 实现自己的播放器。

<!--more-->

## 组件

主要有3个模块

- 视频预览

- 进度条
  - seek

- 播放控件
  - play
  - pause
  - mute on
  - mute off


```html
<div>
<a-player-container ref="videoRef" :width="width" :height="height" @click="autoPlay">
  <a-spin v-show="loading" />
</a-player-container>

<a-progress
  seekable
  :pointer="pointer"
  :currentTime="currentTime"
  :duration="duration"
  @seek="onSeek"
  @dragEnd="onSeek"
></a-progress>

<a-player-control
  :currentTime="currentTime"
  :duration="duration"
  :muted="muted"
  :playing="playing"
  :format="format"
  @pause="pause"
  @play="play"
  @muteOn="openMute"
  @muteOff="closeMute"
/>
</div>
```

## API

定义一个类 `MyVideo`，至少需要暴露以下几个方法

```js
class MyVideo {
  play() {
    if (!this.elem) return

    if (!this.elem.paused) return

    // ready to play for the next frame
    if (this.elem.readyState <= 2) return console.log('not ready 2')

    this.elem.play()
  }
  pause() {
    if (!this.elem) return

    if (this.elem.paused) return

    this.elem.pause()
  }
  openMute() {
    if (!this.elem) return
    this.elem.muted = true
  }
  closeMute() {
    if (!this.elem) return
    this.elem.muted = false
  }
  seek(ms: number) {
    this.elem.currentTime = ms / 1000
  }
  destroy() {
    this.pause()
    if (this.elem) {
      this.elem.src = ''
      this.options.container?.removeChild(this.elem)
      this.elem = null
    }
  }
}
```

### 入参

接下来定义入参，很容易想到，至少要告诉我一个 `url` 吧，其他参数可以是预加载、自动播放、循环播放、事件等和官方 API对齐即可

```js
interface IOption {
  url: string
  container?: HTMLElement // 包含视频的容器
  width?: number // 视频的宽高
  height?: number

  poster?: string // 视频的封面
  preload?: string // metadata, none, auto
  autoplay?: boolean
  loop?: boolean
  muted?: boolean

  onLoadedMetadata?: (duration: number) => void
  onLoadedData?: () => void
  onSeeking?: () => void
  onSeeked?: () => void
  onWaiting?: () => void
  onPlaying?: () => void
  onProgress?: (progress: number) => void
  onPlay?: () => void
  onPause?: () => void
  onAbort?: () => void
  onEnd?: () => void
  onError?: (err: Error) => void
}
```

### 加载视频

这里采用动态加载 `video` 插入到 `DOM` 的方式

```js
class MyVideo {
  async init() {
    this.elem = this.createVideo()

    this.bindEvents()

    this.options.container?.appendChild(this.elem)
  }
  
  createVideo() {
    const video = document.createElement('video')

    video.setAttribute('preload', this.options.preload!)
    if (this.options.poster) {
      video.setAttribute('poster', this.options.poster)
    }
    if (this.options.autoplay) {
      video.setAttribute('autoplay', 'true')
    }
    if (this.options.muted) {
      video.muted = true
    }
    if (this.options.loop) {
      video.setAttribute('loop', 'true')
    }

    video.src = this.options.url
    
    if (this.options.width) {
      video.width = this.options.width
    }
    if (this.options.height) {
      video.height = this.options.height
    }

    return video
  }
}
```

### 定义事件

`Video` 的官方文档就已经提供了很多钩子，我们直接使用接口，比如

```js
class MyVideo {
  bindEvents() {
    this.elem.addEventListener('play', () => {
      typeof this.options.onPlay == 'function' && this.options.onPlay()
    })
  }
}
```

## 预加载内容

`preload` 属性支持

- `auto` 表示视频内容可以被下载，由浏览器决定

- `metadata` 只预加载视频的元信息，比如视频长度

- `none` 视频不会被预加载

因为每个浏览器的默认值不一样，同时在我们的业务里，视频时长是很有用的信息，所以我们默认取  `metadata`


另个预加载的方式是通过使用 `video.load()` 这个方法(加载的内容也是通过 `preload` 属性决定的），不过它更适用于视频的 `src` 发生改变的情况


## timeupdate 更新卡顿

这里遇到一个问题，在 `timeupdate` 通过回调在 `UI` 层更新进度条的时候，会卡顿，原因是 `timeupdate` 的触发频率是 `250ms` 一次 

> The timeupdate event is fired when the time indicated by the currentTime attribute has been updated.
> The event frequency is dependent on the system load, but will be thrown between about 4Hz and 66Hz (assuming the event handlers don't take longer than 250ms to run)


为了解决这个问题，我们需要在 `timeupdate` 里频繁的触发回调，才能达到丝滑般的滚动条前进效果


```js
this.elem.addEventListener('timeupdate', () => {
  this.clearTimer()

  const step = () => {
    // float second
    const currentTime = this.elem ? this.elem!.currentTime * 1000 : 0
    
    typeof this.options.onProgress == 'function' && this.options.onProgress(currentTime)

    this.timer = window.requestAnimationFrame(step)

  }
  step()
})
```

同时需要在其他可能的地方，防止内存泄露，比如

```js
class MyVideo {
  bindEvents() {
    this.elem.addEventListener('pause', () => {
      this.clearTimer()
    })
    this.elem.addEventListener('abort', () => {
      this.clearTimer()
    })
    this.elem.addEventListener('ended', () => {
      this.clearTimer()
    })
    this.elem.addEventListener('error', () => {
      this.clearTimer()
    })
  }
  clearTimer() {
    this.timer && cancelAnimationFrame(this.timer)
    this.timer = null
  }
  destroy() {
    this.clearTimer()
    ....
  }
}
```


## Reference

- [HTMLMediaElement - MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement#instance_methods)
- [timeupdate_event - MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/timeupdate_event)
