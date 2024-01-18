---
title: "Audio Sprite"
date: "2024-01-18"
description: ""
categories: [
  "Frontend",
]
---

## Background

业务中遇到一个 `tts` 生成音频并播放的需求，为了节省 `tts` 生成成本，属于同一个音色的所有文案会生成同一个音频文件，但是前端要按正常的顺序播放。举个例子

```text
0~3 文案一 四川话
3~8 文案二 北京话
8~10 文案三 四川话
10~16 文案四 北京话
```

如上音频长度 `16s`
- 文案一和三是四川话，会生成一个时长 `5s` 的音频文件 `四川话.mp3`
- 文案二和四是北京话，会生成一个时长 `11s` 的音频文件 `北京话.mp3`

所以现在文本音频交叉在各个音频文件中，需求是前端要按正确的顺序播放。

## Audio Sprite
这个问题在游戏开发中很常见，一个小游戏可能就下载了一个声音文件，但其中包含了各种音效比如赢得金币、通关失败、遭受攻击等，然后在必要的时候播放相应的音效，这个技巧就是 `audio sprite`，很像前端的 `css sprite` 只不过是图片而已。对于 `css sprite` 关心的是每个 `sprite` 的位置，对于 `audio sprite` 而言，则是每个 `sprite` 的时间

```ts
{
  src: 'https://xx.mp3',
  sprite: {
    [name]: [startTime, offset, duration],
  }
}
```

- `src` 包含多个音效的一整个音频文件
- `name` 音效的 `id`
- `startTime` 该音效在全局播放器中的开始时间
- `offset` 音效在音频文件中的偏移
- `duration` 该音效的时长

以此，我们可以整理出上文中的 `audio sprite`

```ts
[
  {
    src: 'https://xxx/四川话.mp3',
    sprite: {
      s1: [0, 0, 3000],
      s2: [8000, 3000, 2000] // 单位是 ms
    }
  },
  {
    src: 'https://xxx/北京话.mp3',
    sprite: {
      s1: [3000, 0, 5000],
      s2: [10000, 5000, 6000]
    }
  }
]
```

现在要实现的是按 `startTime` 顺序播放每个 `sprite`，代码如下


```ts
function play() {
  var startTime = Date.now()

  const step = () => {
    const currentTime = Date.now() - startTime

    spriteList.forEach((sprite) => {
      if (sprite.startTime <= currentTime && currentTime <= sprite.startTime + sprite.duration) {
        source.play(sprite.name)
      } else {
        source.pause(sprite.name)
      }
    })
    // 终止播放
    if (currentTime <= totalDuration) {
      requestAnimationFrame(step)
    }
  }

  requestAnimationFrame(step)
}
```

这段代码有几个注意点

- 这里用到了《Game Programming Pattern》提到的模式 `Game Loop`，在 `js` 中，我们使用 `requestAnimationFrame` 实现

- `currentTime` 表示播放器中的当前时间，只要判断 `currentTime` 是否落在 `sprite` 的起止时间内，如果是，播放对应的 `sprite`，否则暂停

- 当 `currentTime` 大于总时长，需要终止播放


## Single Sound

现在的问题是如何播放音频以及播放指定位置的音频 

- 播放音频可以使用 `html5 audio` 或者复杂的 `web audio`
- 指定 `audio.currentTime` 可以实现音频的偏移

整体思路就是

- 每个音频文件创建对应的 `audio` 元素
- 播放 `sprite` 的时候，找到它的时间偏移，设置给 `audio.currentTime`
- 需要注意暂停的时候，要判断暂停的 `sprite` 是不是在播放的 `sprite`，不然会不停触发 `audio.play()/audio.pause()` 因为不同的 `sprite` 共享一个 `audio` 元素


```ts
class Source {
  playing = false
  playingName
  
  constructor(options) {
    this._src = options.src
    this._sprite = options.sprite || {}
    this.load()
  }

  load() {
    this.audioNode = document.createElement('audio')
    this.audioNode.src = src
    document.body.appendChild(this.audioNode)
  }
  pause(name) {
    // 暂停的 `sprite` 是不是在播放的 `sprite`
    if (!this.playing || this.playingName != name) return
    this.audioNode.pause()
    this.playing = false
  }
  play(name) {
    if (this.playing) return
    
    // 找到时间偏移
    const sprite = this.sprite[name]
    this.audioNode.currentTime = sprite[1] / 1000
    
    this.audioNode.play()
    this.playing = true
    this.playingName = name
  }
}
```

不过还有一个问题没解决，所有音频都需要先加载，但只靠 `audio` 标签是没法精准做到的，因为 `audio.src` 并不会立刻加载音频，即使加上 `preload` 属性，也不一定保证100%加载。
其实我们要做的只是加载这个 `mp3` 文件，那就用普通的 `http` 先 `download` 下来，借助于浏览器的缓存 `audio.play()` 的时候，会从缓存中获取。

```ts
function loadAudioFile() {
  const xhr = new XMLHttpRequest()
  xhr.open('GET', url, true)
  xhr.responseType = 'arraybuffer'
  xhr.onload = () => {}
  xhr.send()
}  
```

## Sound Pool

看起来挺简单的，这是因为每个 `sprite` 是按照顺序播放，不会有时间重叠的问题，一个音频文件对应一个 `audio` 元素即可满足。但如果随机随时播放，就会存一个音频文件中不同 `sprites` 同时播放的情况，这就不止有一个 `audio` 元素。

这样，每播放一个 `sprite` 都要创建一个 `audio` 而且在播放完成之后，还要手动移除它，不然页面会有很多 `audio` 元素，内存会很容易被占满。仔细观察，就会发现，反复创建移除是没必要的，相反，可以维护一个 `object pool`，每次生成之前都去 `pool` 里看看有没有空闲的，如果没有才去生成新的。


那问题是谁去维护这么一个 `pool`，自然是每个音频源 `class source`


```ts
class Source {
  _pool: Sound[]

  play(name) {
    const sound = this.findAvailableSound()

    // 找到时间偏移
    const sprite = this.sprite[name]
    sound.playAt(sprite[1] / 1000)
  }
  
  findAvailableSound() {
    let sound = this._pool.find((s) => s.ended || s.paused)
    if (sound) {
      return sound
    }

    return new Sound(this)
  }
}
class Sound {
  id = 0
  
  paused = true
  ended = true
  
  startTime = 0
  endTime = 0
  duration = 0

  audioNode

  constructor(parent: Source) {
    this.audioNode = document.createElement('audio')
    this.audioNode.src = parent._src
    this.audioNode.load()

    this.audioNode.addEventListener('ended', () => {
      this.ended = true
    })
    
    parent._pool.push(this)
  }

  playAt(time: number) {
    this.audioNode.currentTime = time
    this.audioNode.play()
  }
}
```

可以看到和第一版 `class Source` 相比，播放的功能由 `class Sound` 承载，当想要播放某个 `sprite`，就去 `_pool` 里查看一下是否有可用，可用的判断条件是没有声音，即 `paused` 或者 `ended`。

不过当你运行上面的会发现，声音从 `sprite.startTime` 开始播放到整个音频结束，这是因为我们没有判断什么时候该终止这个 `sprite` 的播放。有几种方法解决
- 设置定时器
- 在 `progress` 判断

```ts
class Sound {
  constructor(parent: Source) {
    // method 2
    this.audioNode.addEventListener('progress', () => {
      if (this.audioNode.currentTime >= this.endTime) {
        this.audioNode.pause()
      }
    })
  }
  playAt(time: number) {
    this.audioNode.currentTime = time
    this.audioNode.play()

    // 设置定时器
    setTimeout(() => {
      this.audioNode.pause()
    }, this.duration)
  }
}
```

## Summary

除了 `html5 audio` 之外，还可以用 `web audio` 实现一样的功能，后者功能更为强大，具体实现可以参考 [Howler.js - Github](https://github.com/goldfire/howler.js/tree/master)，上文提到到 `sound pool` 就是出自 `Howler`，源码还有很多地方值得学习的。

## Refer
- [Howler.js - Github](https://github.com/goldfire/howler.js/tree/master)
