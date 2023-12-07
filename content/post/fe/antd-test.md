---
title: "Antd modal has no transition in test env"
date: "2022-01-23"
description: ""
categories: [
    "Frontend",
]
---

记录一下 `antd modal` 在测试环境打开关闭 `modal` 组件没有 `transition` 过渡效果的 bug，说来也是我手欠，改构建改的。

<!--more-->

## bug 复现

业务中使用了 `ant-design-vue`，在使用 `modal` 这个组件的时候，发现测试环境 `modal` 的打开关闭没有 `transition` 动画，但是本地是有的；


## 排查原因

思路一：
有可能是构建没有把相关的 `css` 打包进去？但是打开 `dist` 发现是有的

思路二：
发现是相关的类比如 `fade-enter` 压根就没挂在对应的元素上，那么就是代码本身有问题了。开始单点调试，发现打包后的 `getTransitionProps` 和源码不一致。

这是源码

![](/blog/post/images/getTransitionProps.png)


这是打包后的

![](/blog/post/images/getTransition_dist.png)


这就清楚明了了，跟环境配置有关系，在测试环境部署执行的是 `vite build --mode test` 那么这句话就会把 `NODE_ENV` 设置为 `test` ，这样就导致构建后没有 `transition`


## 解决方法

修改 mode 值 `mode !== test` 即可

