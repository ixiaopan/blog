---
title: "A Cascader Component"
date: "2022-05-29"
description: ""
# tags: []
categories: [
  "Frontend",
]
series: ["Frontend"]
katex: true
---

实现一个业务定制的级联选择器

<!--more-->

## Why

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

显然，这是个树结构

```js
[
  {
    id: "L1",
    label: "level 1",
    children: [
      {
        id: "L2",
        label: "level 1-2"
      }
    ]
  }
]
```


从 UI 来看，切换节点，需要在下一级菜单显示对应的 `children`。如果每次切换，都要遍历一下树，太慢了效率不高，比较好的方式是在一开始就计算好对应的数据，切换的时候从 `map` 里取出来就好。

这里借鉴了文件目录的思想
- 根目录是 `ROOT`

- 查看包含的子文件列表 `$ ls ROOT` 

- 文件所在的目录 `$ cwd`


采用这个思想，那么每个 `node` 的数据结构如下

```js
type INode = {
  id: // the key
  childrenCount: // the number of children
  cwd: // the current working directory
  parent: // 
  depth: // the depth of this node
}
```

接下来，全局维护一个 `map` 对象，其 `key` 即是 `cwd`,

```js
{
  'ROOT': INode[],
  'ROOT/L1': INode[],
  'ROOT/L1/L1-2': INode[],
}
```

最后是深度遍历

```js
treeData = flattenTree(tree, 'ROOT', {}, null)

function flattenTree(list, footpath, treeMap, parent) {
  if (!treeMap[footpath]) {
    treeMap[footpath] = []
  }

  list.forEach(t => {
    t.id = t.id
    t.childrenCount = t.children?.length
    t.depth = splitFootPath(footpath).length
    t.parent = parent
    t.cwd = concatFootPath(footpath, t.id)

    treeMap[footpath].push(omit(t, ['children']))
    
    flattenTree(t.children, t.cwd, treeMap, t)
  })
}
```

### 层级联动

接下来就好办了，监听节点的 `click` 事件，这里要注意

- 可以选择中间层的，点击了 `radio/checkbox` 才会被选中；点击文本，只是展示下一层几

是否只能选择叶子节点，我们借鉴 `checkbox` 的设计，新增属性 `intermediate` 

- 如果是叶子节点，直接选择完成

- 这里通过 `classList` 判断是否点击了 `radio/checkbox`，if so, 直接选择完成

- `switchLevel(record, true)` 通过添加标识(第2个参数 `true`) 标记 选择完成

```js
function rowClick(record, e) {
  if (props.intermediate) {
    // 如果是叶子节点，点自己也是可以的
    // 选择了 radio/checkbox，新增 `true` 表示选择结束
    if (
      !record.childrenCount ||
      (e.target && e.target.classList.contains('wxp-cascader-list-option-radio'))
    ) {
      e.stopPropagation()
      radioCheckedNode = record
      switchLevel(record, true)
    }

    // 点击的是文本，展开下一层
    else if (record.childrenCount) {
      radioCheckedNode = null
      switchLevel(record)
    }
  } else {
    switchLevel(record)
  }

  emit('click', record)
}
```

层级的切换，有几个情况

- 向更深的层级展开

- 在本层级不同兄弟节点之间

- 回到父层级


这也很好办，结合当前路径 `currentPath` ，所选节点的信息比如 `cwd, depth`，就可以知道是是哪个情况


```js
function switchLevel() {
  // 向更深的层级展开
  if (currentPath.value.length == record.depth) {
    currentPath.value = currentPath.value.concat(record.cwd)
  } else { // 切换到上一级，或者本层级
    const temp = currentPath.value.slice(0, record.depth)
    currentPath.value = temp.concat(record.cwd)
  }
}
```

最后就是渲染数据了，监听 `currentPath` 重新计算 `currentList` 即可

```js
// currentPath: ['ROOT', 'L1', 'L1-2']
nextList = currentPath?.map((fp, idx) => {
  return treeMap[fp]?.map(ot => {
      const t = { ...ot } // copy
      
      // UI样式相关的逻辑
      const selecting = t.cwd == currentPath.value[idx+1]
      const selectingEnded = soleEnded.value && selecting
      const selectingEndedLeaf = props.intermediate ? t.cwd == radioCheckedNode?.cwd : !t.childrenCount && selectingEnded

      return {
        ...t,
        selecting,
        selectingEnded,
        selectingEndedLeaf,
      }
    })
})

if (props.intermediate && radioCheckedNode) {
  currentList.value = nextList.slice(0, radioCheckedNode.depth)
} else {
  currentList.value = nextList
}
```



### 节点互斥

前面提到，节点之间存在互斥行为

- 通用节点，权限最高，与所有其他节点互斥

- 选择了父节点，其孩子节点都不可选


不可选，那就是 `disabled`， 在上面的 `nextList = ...` 中新增 `disabled` 的判断即可


针对通用节点（该节点只有一个），我们新增属性 `majestyId` (your majesty 女王陛下)，由业务指定节点 `id`

```js
disabled = (majestySelected.value && (item.id !== props.majestyId))
```

针对第二个情况，遍历每一层的节点 `item`，如果

- 当前层级深度比选择的节点所在的层级深，说明当前渲染的是孩子节点

- 然后判断 `item` 是否依赖选择的节点 `t`，即是否有 "血缘关系" 


```js
disabled = !!(nextTagList?.find((t) => t.depth < item.depth && hasParentDependency(item, t)))
```

### 搜索筛选

最后是搜索的实现(前端搜索，因为业务数据量不大)，也分两种情况

- 只在叶子节点搜

- 对整棵树进行搜索，且搜索结果需要按深度层级显示(换句话说，也是深度遍历)
  
  - `Node_L1_1` 匹配，继续匹配孩子节点
    
    - `Node_L1_1_L2_2` 匹配，继续匹配孩子节点
    
      - `Node_L1_1_L2_2_L3_3`匹配 ，继续匹配孩子节点
    
    - 如果 `Node_L1_1_L2_6` 匹配，继续匹配孩子节点
      

  - `Node_L1_2` 匹配，继续匹配孩子节点

  - ...


#### 叶子搜索

叶子节点搜索比较简单，首先要搜集所有的叶子节点，在 `flattenTree()` 中即可以实现

```js
// 缓存所有叶子节点
if (!t.childrenCount) {
  leafNodeList.push({ ...(omit(t, ['children'])) })
}
```

然后根据关键字过滤一下即可

```js
queryList = leafNodeList.filter(t => t.label.indexOf(q) > -1)
```

#### 路径搜索


既然知道了叶子节点( `flattenTree()` 的深度遍历和这里层级搜索是一一对应的，所以按顺序遍历叶子节点就是我们所需要的路径顺序)，那么我们可以通过 `parent` 回溯对应的链路

思考下面的路径

```js
[A, B, C, D1]
[A, B, C, D2]
```

最终所有的路径顺序

```js
[A]
[A, B]
[A, B, C]
[A, B, C, D1]
[A, B, C, D2]
```

这一步也简单，不断找 `parent` 就好了

```js
function reachMe(node) {
  let tempList = [ node ]
  let o = node.parent
  while (o) {
    tempList.unshift(o)
    o = o.parent
  }
  return tempList
}
```
当给定一个节点，比如说 `D1` ，这个函数就会返回 `[A, B, C, D1]`，所以要得所有路径，还需要在遍历 `D1.parent` 直到 `null`



这里要注意一个问题，当我们继续遍历 `D2` 的时候，`[A], [A, B], [A, B, C]` 已经存在了，所以要去重。去重也很简单，只需要看一下最后一个节点是不是当前节点就行，

- 比如 `D2`，这是肯定没有的，所以需要计算；

- 但是到了`D2.parent` 即 `C` 的时候，`memo` 里已经存在 `[A, B, C]` 了，所以不用再算了


```js
oneLevelList = leafNodeList.reduce((memo, node) => {
  let resultList = []
  
  let o = node
  while (o) {
    if (!memo.length || !memo.find(nl => nl[nl.length - 1]['id'] == o.id)) { 
      resultList.unshift(reachMe(o))
    }
    o = o.parent
  }

  memo = memo.concat(resultList)

  return memo
}, [])
```


最后就是搜索匹配，类似的，只需要匹配每条路径的最后一个节点即可

```js
if (props.intermediate) {
  // [[p1], [p1, p2], [p1, p2, leaf]]
  nextList = oneLevelList.filter((res) => {
    const lastNode = res[res.length - 1]
    return lastNode.label.indexOf(q) > -1
  })
}
```
