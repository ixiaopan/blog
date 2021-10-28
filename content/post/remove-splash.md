---
title: "REMOVING ANNOYING SPLASH ADs!!!"
date: "2021-10-28"
description: ""
# tags: []
categories: [
    "Utilities",
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



Furthermore, we can even modify responses. `script-response-body` enables us to write javascripts to decide what content to show. It's very useful to filter misc data appearing on the page, such as promotions, egg hunts or ads. For example, I wrote a simple `Zhihu.js` to change the page mode from mobile to PC and remove the annoying login modal since you cannot use Zhihu unless you download their app.

```js
// PC version of zhihu: remove login modal and guidance for downloading app
let body = $response.body

identifier = '</script></head>'
insert_pos = body.indexOf(identifier)

work = 'setTimeout(function(){ const openInAppButton = document.querySelector(".OpenInAppButton"); if (openInAppButton) { openInAppButton.style.display = "none"; } const closeBtn = document.querySelector(".Modal-closeButton"); if (closeBtn) { closeBtn.click() }; const btns = document.querySelectorAll(".ModalExp-modalShow .ModalWrap-itemBtn"); if (btns && btns.length && btns[1]) { btns[1].click(); }}, 2500)';
body = body.slice(0, insert_pos) + work + body.slice(insert_pos)

$done({ body: body })

```



## Get Free From it



I have sort out several rules for commonly used APPs, well, just for my personal use. Hopefully, it can help you to get free from the annoying splash ads.



```bash
# Keep
^https://api\.gotokeep\.com/(op-engine-webapp|ads)/v\d/ads?/preload url reject-dict
^https?://api\.gotokeep\.com/op-engine-webapp/v\d/ads? url reject-dict
^https://store\.gotokeep\.com/api/v\d/mypage/egg url reject-dict
^https?://api\.gotokeep\.com/homepage/v\d/tab\? url script-response-body Keep.js
^https://api\.gotokeep\.com/athena/v\d/people/my$ url script-response-body Keep.js
^https://api\.gotokeep\.com/config/v\d/basic url script-response-body Keep.js

# Bilibili
^https://api\.bilibili\.com/x/v\d/dm/ad url reject-dict
^https://app\.bilibili\.com/x/v\d/splash/ url reject
^https?://manga\.bilibili\.com\/twirp\/comic\.v\d\.Comic\/Flash url reject
# keep '?' to distinguish 'index/story/'
^https?://app\.bilibili\.com/x/v\d/feed/index\? url script-response-body bili_feed.js
^https?://app\.bilibili\.com/x/resource/show/tab/ url script-response-body bili_feed.js
^https?://app\.bilibili\.com/x/v\d/splash/list url script-response-body bili_feed.js
^https://app\.bilibili\.com/x/v\d/account/mine url script-response-body bili_feed.js


# Douban
^https?:\/\/api\.douban\.com\/v\d\/app_ads\/ url reject

# zhihu
^https://www.zhihu.com/api/v\d/market/rhea/questions/.+/qa_related url reject
^https?:\/\/www\.zhihu\.com\/commercial_api\/banners_v3\/question_down_sticky url reject-dict
^https?:\/\/www\.zhihu\.com\/commercial_api\/banners_v3\/question_up url reject-dict
^https://www\.zhihu\.com/ url request-header (\r\n)User-Agent:.+(\r\n) request-header $1User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.81 Safari/537.36$2
^https://.+?\.zhihu\.com/ url script-response-body zhihu.js


# Baidu Tieba
^https?:\/\/c\.tieba\.baidu\.com\/\w+\/\w+\/(sync|newRnSync|mlog) url reject
^https?:\/\/c\.tieba\.baidu\.com\/c\/p\/img\?src= url reject-img
^https?:\/\/c\.tieba\.baidu\.com\/c\/s\/logtogether\?cmd= url reject-img
^https?:\/\/c\.tieba\.baidu\.com\/c\/s\/splashSchedule url reject

# renren
^https?:\/\/api\.rr\.tv\/ad\/ url reject
^https?:\/\/api\.rr\.tv\/.*?(getAll|Version) url reject
^https://api\.rr\.tv/v3plus/index/channel\?pageNum=1&position=CHANNEL_MY url script-response-body rr.js
^https?://.+?\.pglstatp-toutiao\.com/ url reject
^https?://pgdt\.ugdtimg\.com/ url reject
^https?://blink-upload\.bayescom\.com/ url reject

# ctrip
^https://m\.ctrip\.com/restapi/soa2/13916/scjson/tripAds url reject-dict


# netease mail
^https?://.+?\.ws.126.net/ url reject

```

