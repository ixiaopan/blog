---
title: "REMOVING ANNOYING SPLASH ADs!!!"
date: "2021-10-28"
description: ""
# tags: []
categories: [
    "Computer Science",
]
series: ["Computer Science"]
katex: true
---



## What and Why

I don't know when it happened, but without a doubt, you will see a statistic or dynamic image on your screen when you launch some APPs nowadays (especially if you are somewhere on the Earth). The picture looks like the figure below with a small button to enable people to skip ads. Generally, the showing time of ads lasts at least 3 seconds. The most disgusting thing is that there is no upper bound for this, so you have to suffer from this unless you don't use any app at all.



<img src="https://static1.keepcdn.com/ark_optimus/2021/10/27/1635333663953ap3s8jls_750x1334.jpg" style="width:30%"/>





Technically speaking, the first page you see when you open an APP is known as the splash page, which originated from the comics industry. In the beginning, they were used as a poster to explain the whole story to readers. Similarly, the purpose of splash pages in mobile applications is to enhance the impression on people.  But now, splash pages are abused as a means of earning advertising profits by commercial companies, getting far away from the original intention.



## Under the hood

Before getting rid of them, we should know how they are implemented. After several hours hard working, I figured out some key points:



- images for ads are usually hosted on some special statistic servers
- there are specific APIs for advertising data 
- `ad/adver/preload/splash...` are common words to identify ads
- we can control some behaviors of loading ads, such as whether to preload, play in wifi only, duration, etc. by modifying the metadata of ads that returned through APIs



Since they are both URIs, we simply reject these requests or response with an empty JSON. With the aid of some proxy tools (Quantumult X), we can rewrite the request as follows,

```js

^https://api\.gotokeep\.com/(op-engine-webapp|ads)/v\d/ads?/preload url reject-dict
^https?://api\.gotokeep\.com/op-engine-webapp/v\d/ad url reject-dict
^https://store\.gotokeep\.com/api/v\d/mypage/egg url reject-dict
^https?://api\.gotokeep\.com/homepage/v\d/tab url script-response-body Keep.js
^https://api\.gotokeep\.com/athena/v\d/people/my url script-response-body Keep.js

```



Furthermore, we can even modify responses. `script-response-body` enables us to write javascripts to decide what content to show. It's very useful to filter misc data appearing on the page, such as promotions, egg hunts or ads. For example, I wrote a simple `Zhihu.js` to change the page mode from mobile to PC and remove the annoying login modal, since you cannot use Zhihu unless you download their app.

```js
// PC version of zhihu: remove login modal and guidance for downloading app
let body = $response.body

identifier = '</script></head>'
insert_pos = body.indexOf(identifier)
if (insert_post>-1){
  work = 'setTimeout(function(){ const openInAppButton = document.querySelector(".OpenInAppButton"); if (openInAppButton) { openInAppButton.style.display = "none"; } const closeBtn = document.querySelector(".Modal-closeButton"); if (closeBtn) { closeBtn.click() }; const btns = document.querySelectorAll(".ModalExp-modalShow .ModalWrap-itemBtn"); if (btns && btns.length && btns[1]) { btns[1].click(); }}, 2500)';
  body = body.slice(0, insert_pos) + work + body.slice(insert_pos)
}
$done({ body: body })

```



## Get Free From it



I have sort out several rules for commonly used APPs — just for personal use  (You can find it on my [Github](https://github.com/ixiaopan/DataScience/tree/master/Utilities/quantumult)). Hopefully, it can help you get free from the annoying splash ads.

