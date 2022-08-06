---
title: "Video autoplay not working when low power mode is on (iphone)"
date: "2022-08-06"
description: ""
# tags: []
categories: [
  "frontend",
]
series: ["frontend"]
katex: true
---

An interesting bug about the HTML5 video.

<!--more-->

Last week, I inserted a video element in a page. As you can expect, the video is muted and autoplay when page is loaded. 

```html
<video src="/trailer.mp4" loop muted autoplay playsinline />
```

Unfortunately, it didn't play when I opened it using safari. Technically speaking, there is nothing wrong with the above code snippet. So, what's the problem? 

Thanks to StackOverflow, I found a clue.

> I had same issue with apple devices like iPhone and iPad, I turned off the low power mode....

The reason is that I turned on the low power mode, emmm...


## Refer
- https://stackoverflow.com/questions/20347352/html5-video-tag-not-working-in-safari-iphone-and-ipad
