---
title: "A cascader component"
date: "2022-05-29"
description: ""
# tags: []
categories: [
  "frontend",
]
series: ["frontend"]
katex: true
---

实现一个业务定制的级联选择器

<!--more-->

## Background

常见的级联选择器，比如省市区选择器，实现起来比较简单，而且父子之间没有过多的依赖关系。然而由于我们业务的特殊性，这些选择器根本无法满足需求，不得已只能从0到1实现。先梳理一下要实现的功能

- 支持单选、多选
  - 单选即只能选择一个子项，又分为两种情况
    - case 1, 只能选择叶子节点
    - case 2, 可以选到任一个层级
  - 多选 case 3
    - 可以选到任一个层级，可以选择多个项
    - 选择了通用节点，所有其他节点都不能被选择，即互斥关系
    - 选择了父节点，其所有孩子节点都不能被选择，但是依然可以访问其路径

- 支持筛选
  - 只支持叶子节点筛选，适用于 case 1 (单选且只能选择到叶子节点)
  - 所有层级都支持筛选，使用 case2 & case 3

效果如下，

单选
![](/blog/post/images/cascader-sole.gif)

多选
![](/blog/post/images/cascader-mul.gif)


## 实现

### 数据格式


### 层级联动

### 节点互斥

### 搜索筛选

