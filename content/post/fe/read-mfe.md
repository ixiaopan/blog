---
title: "《Micro Frontends in Action》"
date: "2024-03-18"
description: ""
tags: [Reading]
categories: [
  "Frontend",
]
---

## Micro Frontend

### what

> Micro frontends are not a concrete technology. They’re an alternative organizational and architectural approach for scaling development.

- Teams can work autonomously in their field of expertise.
- Teams can choose the technology stack that fits best for the job at hand.
- The applications are loosely coupled and only integrate in the frontend (e.g., via links).

### how

![](/blog/post/images/mfe-01.png)

`routing` 指页面之间的 `navigation`

- 简单的方式，通过 `<a>` 跳转，问题是会刷新页面
- 如果是 `spa`，一般是通过自建 `application shell` 或者三方框架，比如 `single-spa`

`composition` 即页面内组件的组装

- client-side: 比如 `iframe/ajax/web components`
- server-side: 比如 `ssi/esiTailor/Podium`

`communication`，即页面、组件之间的数据通信

### misc

#### web performance

因为不同页面或系统是不同的团队来维护，这很可能造成代码冗余、引入了同一框架的不同版本等问题

#### shared design system

最好不同系统之间保持统一的 UI 样式，给用户良好的用户体验

#### shared knowledge

前端底层工具等不要重复开发，应该在大团队内共享，比如日志、监控、utils等。但是 sharing 也是有成本的，也要考虑跨团队合作，比如升级了日志系统，要通知相关团队进行测试等。

一种方案是 share nothing，虽然有冗余，但对于 mfe 这种要求 autonomy 以及更快的迭代的架构来说是可接受的。


### pros&cons

pros
- autonomy
- higher iteration speed
- free technology choice
- support legacy system

cons
- redundancy，比如一个框架出现了问题，所有团队都要升级版本
- consistency，理想情况下，每个团队的数据是独立的，但是实际上，teamA可能会依赖teamB的数据，这就要考虑数据的一致性
- 每个子系统是一样的技术栈还是不同的技术栈？相同技术栈可以共享开发经验、基础建设等减少代码冗余，不同技术栈切换存在成本


### setup

- find team boundaries
- establish common rules that all teams agree on, such as namespaces
- which team is responsible for performance issue
- build shared service and infras
- ...


## 原理

routing
- `<a href>`
- client routing via app shell

composition
- iframe
- ajax
- web component

communication
- UI communication
- Frontend-backend
- data replication


### iframe
```html
<aside class="recos">
  <iframe src="http://localhost:3002/recommendations/eicher" />
</aside>
```

### ajax

```ts
<aside
  class="decide_recos"
  data-fragment="http://localhost:3002/fragment/recommendations/eicher"
>
  <a href="http://localhost:3002/recommendations/eicher">
    Show Recommendations
  </a>
</aside>
<script src="/static/page.js" async></script>

// /static/page.js
(function() {
  const element = document.querySelector(".decide_recos");
  const url = element.getAttribute("data-fragment");

  window
    .fetch(url)
    .then(res => res.text())
    .then(html => {
      element.innerHTML = html;
    });
})()
```

#### isolating css
- team prefix
- css modules、less、saas
- css-in-js


#### isolating js
- IIFE
- namespace in global scope
  - localstorage/cookies
  - `window.teamA.xxx, window.teamB.xxx`

#### pros/cons
pros
- integrate all content into one DOM
- progressive enhancement
- flexible error handling

cons
- async
- lacking isolation (css/js)
- server request required
- no lifecycle for scripts


### web component

```ts
<aside class="decide_recos">
  <inspire-recommendations sku="eicher"></inspire-recommendations>
</aside>
<script src="http://localhost:3002/static/fragment.js" async></script>


// fragment.js
const prices = {
  porsche: 66,
  fendt: 54,
  eicher: 58,
};

class CheckoutBuy extends HTMLElement {
  connectedCallback() {
    const sku = this.getAttribute("sku");
    this.innerHTML = `
      <button type="button">buy for $${prices[sku]}</button>
    `;
    this.querySelector("button").addEventListener("click", () => {
      alert("Thank you ❤️");
    });
  }
  disconnectedCallback() {
    this.querySelector("button").removeEventListener("click");
  }
}
window.customElements.define("checkout-buy", CheckoutBuy)
```

#### isolating css

`shadow dom` 自带隔离属性

```ts
class CheckoutBuy extends HTMLElement {
  connectedCallback() {
    this.attachShadow({ mode: "open" });
    
    this.shadowRoot.innerHTML = `
      <style>
        button { background: rgb(75, 158, 204); }
      </style>
      <button type="button">buy for $${prices[sku]}</button>
    `;
  }
}
```

### communication

- page-to-page: via simple links
- within one page
  - parent to fragment
  - fragment to parent
  - fragment to fragment

#### parent to fragment

pass props

```ts
// decide.js
const buyButton = document.querySelector("checkout-buy")
platinum.addEventListener("change", e => {
  const edition = e.target.checked ? "platinum" : "standard"
  buyButton.setAttribute("edition", edition)
})

// checkout.js
class CheckoutBuy extends HTMLElement {
  attributeChangedCallback() {
    this.render();
  }
}
```
#### fragment to parent
emit custom event

```ts
// decide.js
const buyButton = document.querySelector("checkout-buy");
buyButton.addEventListener("checkout:item_added", e => {
  element.classList.add("decide_product--confirm");
});

// checkout.js
class CheckoutBuy extends HTMLElement {
  render() {
    this.querySelector("button").addEventListener("click", () => {
      this.dispatchEvent(new CustomEvent("checkout:item_added")) // emit event
    });
  }
}
```

#### fragment to fragment
- direct 
- via their parent
- pub/sub

```ts
// inspire.js
window.addEventListener('hello', () => {})

// checkout.js
fragmentA.dispatchEvent(new CustomEvent('hello', { bubbles: true, payload: {} }))
```

使用 `broadcast channel api` 实现 pub/sub

```ts
// inspire.js
const channel = new BroadcastChannel("tractor_channel"); 
channel.onmessage = function(e) { 
  if (e.data.type === "checkout:item_added") { 
     console.log(`tractor ${e.data.type} added`)
  }
}

// checkout.js
const channel = new BroadcastChannel("tractor_channel")
const buyButton = document.querySelector("button")
buyButton.addEventListener("click", () => {
channel.postMessage({type: "checkout:item_added", sku: "fendt"}
```

### app shell

> Hard navigation describes a page transition where the browser loads the complete HTML for the next page from the server.
> 
> Soft navigation refers to a page transition that’s entirely client-side rendered, typically by using a client-side router. In this scenario the client fetches its data via an API from the server.

![](/blog/post/images/mfe-2.png)

![](/blog/post/images/mfe-3.png)

#### what

`app shell` 监听 url change 然后渲染对应的 mfe，此外 `app shell` 可能还需要

- context information (language, userinfo)
- authentication
- error handling
- performance monitoring

#### flat routing

- app shell 要知道路由、组件的映射
- 组件之间需要知道 url，如果要实现组件间跳转

```ts
// app-shell.html
<div id="app-shell">
  <div id="app-content"><span>content not loaded yet</span></div>
</div>

// app-shell.js
const routes = {
  "/": "inspire-home",
  "/checkout/pay": "checkout-pay",
}
function updatePageComponent(pathname) {
  const comp = document.createElement(routes[pathname])
  appContent.replaceChild(comp, appContent.firstChild)
}
// 三方 history 实现
myHistory.listen(updatePageComponent)
updatePageComponent(window.location)

// 
document.addEventListener("click", e => {
  if (e.target.nodeName === "A") {
    const href = e.target.getAttribute("href");
    myHistory.push(href)
    e.preventDefault()
  }
})
```

#### two-level routing

单层路由的问题在于 app shell 要维护所有 url，实际上只需要子系统的路径，系统内的跳转各自负责即可

`app-shell` 层和单层路由的代码一样，只是 `routes map` 只有顶级路由映射了 

```ts
// app-shell.js
const routes = {
  "/": "inspire-pages"
  "/product/": "decide-pages",
  "/checkout/": "checkout-pages",
}
```

以 `checkout` 为例，页面地址为 `/checkout/pay` 时，代码运行如下

```ts
// checkout.ts
const routes = {
  "/checkout/pay": () => `<a href="/checkout/success">buy now</a>`,
  "/checkout/success": () => `<h1>Success</h1>`
};
class CheckoutPages extends HTMLElement {
  connectedCallback() {
    this.render(window.location)
    this.unlisten = myHistory.listen(location => this.render(location))
  }
  render(location) {
    const route = routes[location.pathname]
    if (route) {
      this.innerHTML = route()
    }
  }
  disconnectedCallback() {
    this.unlisten()
  }
}
window.customElements.define("checkout-pages", CheckoutPages);
```

## performance

### assets loading

### performance

## 框架

### single-spa

### qiankun

### micro-app


