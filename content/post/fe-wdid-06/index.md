---
title: "WDID - The track panel of a video editor"
date: "2023-06-26"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---

In the previous post, we have learned how to design a video player. In fact, a video player is a container containing many tracks, such as bgm, text, and so on. Can we show and edit these tracks? Of couse, we can. Today, we will introduce the basic UI implmentation of the track panel of a video editor.

<!--more-->

## Comps

As shown in the above image, we can see the track panel has three main parts:
- timeline(scale + indicator)
  - the container for both zoom and track


- zoom
  - zoom-in and zoom-out, which affects the width of the timeline and the track
  - the initial reference is called `baseWidth`, which means how many pixels 1 second hold
  - When zoom in, `baseWidth` becomes larger; When zoom out, `baseWidth` becomes smaller


- track
  - several types, e.g. `bgm/text/tts...`
  
  - each track type can have many segments, and segments can overlap. If so, we move down the last segment like down staircases. We use `yIndex` indicates the index of the segment along the `y` axis in the same track type. Usually, `yIndex` is `0`.
  
  - each track has its start position defined by offsetX(`startTime * baseWidth`) and offsetY(`yIndex*trackHeight`) as well as width decided by `duration` and `baseWidth`.


## Track

First, different track type has different styles, so we need to define each track type first.

```js
enum TRACK_TYPE {
  BGM = 1,
  TEXT,
  TTS,
  SUBTITLE,
  VIDEO,
  CLIP,
}
```

Second, each track has its position and width, which is determined by the start time, duration, yIndex and the baseWidth.

```js
width = duration * baseWidth

offsetX = startTime * baseWidth

offsetY = yIndex * someHeight
```


```vue
<template>
  <div :class="['track', 'track-' + type]" :style="styleObj" :data-scale="baseWidth" :data-start="startTime" :data-duration="duration">
    <div :class="['track-inner']">
      <icon />
      <span class="track-text"><slot /></span>
    </div>
  </div>
</template>

<script>
const styleObj = computed(() => {
  return {
    width: (props.baseWidth / 1000 * props.duration) + 'px',
    transform: 'translate(' + (props.baseWidth / 100 * props.startTime) + 'px,' + props.yIndex * props.yOffset + 'px)',
  }
})
</script>

<style>
.track-bgm {
  &:hover .track-inner {
    outline: 1px solid #f00;
    outline-offset: -1px;
  }
  &:active .track-inner,
  &.active .track-inner {
    outline: 2px solid #f00;
    outline-offset: -2px;
  }
}
</style>
```

The interesting thing here is the use of `outline`.

## Zoom


## Timeline
