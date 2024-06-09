---
title: "CSS Review"
date: "2024-03-09"
description: ""
categories : [
  "Frontend",
]
---

## box model

css 把文档树中的每个元素描述为一个 box，通过相关的属性决定每个 box 的大小、位置、在画布上的层叠顺序 stacking level等。

每个 box 都是一个矩形框，包括
- content area
- padding area
- border area
- margin area

## padding
- padding 不能为负值
- 百分比是相对元素的包含块的宽度

## border
- border-color 默认取 color 属性的值
- border-left 会重置 width/style/color 的值，这意味着如下代码中的 `p` 的 `left border` 是 black，不是 red

```css
p {
  border: solid red;
  border-left: double;
  color: black;
}
```

## margin

- margin 可以是负值，比如常见的 『负magin 布局』
- 百分比是相对元素的包含块的宽度
- 对于 non-replaced inline element 比如 `span/em/strong`，设置 margin-top/bottom 无效
- collapsing margins 

### collapsed margin

1、什么是 collapsed margin

- 相邻2个 box 之间（不一定是兄弟元素，父子之间也可以）的 margin 会合并为一个 margin，即 collapsed margin

- 水平方向上的 margin 不会发生 collapse

2、为什么会折叠

可以参考 [Stackoverflow](https://stackoverflow.com/questions/3069921/what-is-the-point-of-css-collapsing-margins) 上这个回答

> The general meaning of “margin” isn’t to convey “move this over by 10px” but rather, “there must be 10px of empty space beside this element.” I’ve always found this is easiest to conceptualize with paragraphs. 
>
> If you just gave paragraphs margin-top: 10px and had no margins on any other elements, a series of paragraphs would be spaced beautifully. But of course, you’d run into trouble when placing another element underneath a paragraph. The two would touch.
>
> If margins didn’t collapse, you’d hesitate to add margin-bottom: 10px to your previous code, because then any pair of paragraphs would get spaced 20px apart, while paragraphs would separate from other elements by only 10px.
>
>So vertical margins collapse. By adding top and bottom margins of 10px you’re saying, “I don’t care what margin rules any other elements have. I demand at least 10px of padding above and below each of my paragraphs.”

3、什么时候会折叠

满足以下条件
- both belong to in-flow block-level boxes that participate in the same block formatting context
- no line boxes, no clearance, no padding and no border separate them
- 垂直方向相邻
  - top margin of a box and top margin of its first in-flow child （父、第一个孩子相邻）
  - bottom margin of box and top margin of its next in-flow following sibling（兄弟相邻）
  - bottom margin of a last in-flow child and bottom margin of its parent if the parent has 'auto' computed height （父高度是auto height、最后一个孩子）
  - top and bottom margins of a box that does not establish a new block formatting context and that has zero computed 'min-height', zero or 'auto' computed 'height', and no in-flow children （空元素）

4、怎么折叠

- 对于正的边距，取最大的值
- 对于一正一负，从正的一方减去负的绝对值
- 对于都是负值的，从0中减去绝对值最大的

5、如何规避折叠

- Margins between a floated box and any other box do not collapse, not even between a float and its in-flow children
- Margins of absolutely positioned elements do not collapse, not even with their in-flow children
- Margins of inline-block elements do not collapse, not even with their in-flow children
- elements that esablish a new bfc（如overflow 值不是 visible、float、绝对定位等）do not collapse with their in-flow children
- 父、第一个孩子相邻
  - 父元素添加 padding、border
  - child has clearence
- 兄弟相邻
  - sibling has clearence

## References
- [CSS3 Box Model - W3C](https://www.w3.org/TR/css-box-3/)
- [CSS2 Box Model - W3C](https://www.w3.org/TR/CSS2/box.html)
