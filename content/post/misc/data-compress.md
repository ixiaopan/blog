---
title: "《数据压缩入门》"
date: "2023-10-25"
description: ""
tags: [Reading]
categories: [
  "Misc",
]
katex: true
---


前几天得闲随手翻了翻 [《数据压缩入门 - 豆瓣》](https://book.douban.com/subject/35034359/)，内容是比较浅显，但是真的要深入理解并应用这些算法还是有一定难度，印象比较深刻的是这3个算法

- 哈夫曼编码
- 算术编码
- LZ编码

在实际应用中，这几个算法也是经常结合使用的，比如 `gzip`。

<!--more-->

## 压缩思路

- 减少数据集中不同符号的数量，比如 `TO BE OR NOT TO BE` 如果按字符级别就有 `TOBERN` 6个符号，而按单词只有 `TO` `BE` `OR` `NOT` 4个，如果再按 `TO BE` `OR` `NOT` 只有3个

- 用更少的位数对更常见的符号进行编码，这个道理就显而易见的了

但是也要考虑其他因素

- 不同数据类型处理方法不同

- 有些数据需要转换才能压缩

- 数据可能是偏态的，比如温度夏天偏高冬天偏低


## Entropy

给定任意一个十进制整数，用二进制来表示这个数最少需要多少二进制位

$$
log_2(x) = ceil(log(x + 1) / log(2))
$$

这个值就是熵（entropy），扩展到整个数据集

$$
H(S) = - \sum_{i=1}^{n} p_i log_2(p_i)
$$

从上面的公式可以看出，
- 熵是建立在对每个符号出现概率的估算之上的
- 熵表示的是数据集中的每个符号平均所需的最小二进制位数
- 一个符号出现得越频繁，$p_i$ 越大，熵越小，即越不重要

有个在线计算 [shannon entropy](http://www.shannonentropy.netmark.pl) 的网站，比如我们输入 `ABBCCCDDDD` 

首先会先计算频率
A: 0.1
B: 0.2
C: 0.3
D: 0.4

然后根据公示计算熵

$$
H(S) =  -[(0.1log_2 0.1)+(0.2log_2 0.2)+(0.3log_2 0.3)+(0.4log_2 0.4)] = 1.84644
$$

这个熵表示每个符号平均需要 `2bit` 表示，整个字符串也就是 `20bit`


### 突破熵

上面公式有个明显的问题 —— 只考虑了概率，忽略了数据本身含有的结构信息，比如顺序，语义等。比如[1,2,3,4]和 [3,2,1,4] 虽然熵一样，但是前者是线性递增序列，明显可以用更少的信息存储。


所以我们可以利用数据集的结构信息将其转换为一种新的表示形式，这也是突破熵的关键，可行的方法比如

- 增量编码 delta coding：将数转换为与上一个数的差，比如 [0,1,2,3,4,5,6,7] 转换为[0,1,1,1,1,1,1,1]

- 字符分组


## VLC

需要寻找合适的码字，符合数据中符号的出现概率分布。


### 前缀性质

如何用 VLC 编码？

- 计算数据集中每个符号的出现概率

- 根据概率为每个符号分配码字，一个符号出现的概率越大，所分配的码字就越短

- 再次遍历数据集，对每一个符号进行编码，并将对应的码字输出到压缩后的数据流中

如何解码？

假设现在有如下编码 `A: 0, B: 10, C: 101, D: 111`，有待解码字符串 `0101111`，

首先 `0` 对应 `A`，然后 `1` 没有找到，继续得到 `10` 发现是 `B or C` 继续输入得 `101` 是 `C`，但是这里就有歧义了，`101` 是 `10` + `1` 即 `B+?` 还是就只是 `101` 即 `C` 

从这个例子可以看出，如果一个码字被分配了，就不能再用作另个码字的前缀，即码字需要满足 `前缀性质`

### 一元码

任意正整数，`n`个  `1` 或 `0` 后面跟着一个 `0` 或者 `1`。比如 

| number  | unary code | alternative | probability |
|--------| ---------|--------|----|
| 0  |  0 | 1 |   |
| 1 | 10 | 01 | 0.5 |
| 2 | 110 | 001 | 0.25 |
| 3 | 1110 | 0001 | 0.125 |

- 本质就是有些位数 `1` 当作值来使用，而最后一位 `0` 作为分隔符使用

- 一元码满足前缀性质

- 但是对于大数字，比如 `1000` 那就是有 1000个1 加上 1个0，这就有点过犹不及了。


### Elias Gamma

适用于无法确定上限的整数编码，分解 `x` 为2个因子，分别用一元码和二进制编码表示，同时为了保证可以无歧义的解码，二进制编码的长度是一元码的长度。

$$
x = 2^e + d
$$


举个例子， `42 = 2^5 + 10` 即一元码是 `111110`，二进制是 `1010`，合在一起就是 `111110:01010`

cons
- 因为使用了一元码，对于大数字，要经过位数很长的编码，比如1000要19个bit

```ts
function egc(n) {
    const u = Math.floor(Math.log2(n))
    const b = n - Math.pow(2, u)
    
    const us = new Array(u).fill(1).join('')
    let bs = b.toString(2)
    if (bs.length < us.length) {
        bs = bs.padStart(us.length, '0')
    }
    console.log('bit', us + '0' + bs)
    return us.length * 2 + 1
}
```

举个例子，有如下数据 `10, 990, 21, 1`

- 如果使用固定编码，需要 `10bit * 4 = 40bit`

- 使用 `Elias Gamma` 需要 `egc(10) + egc(990) + egc(21) + egc(1) = 36bit`，少了 `4bit`



## 统计/熵编码

给定一个数据集，我们需要找到和符号出现概率匹配的VLC方法，从而为其分配码字，但不是每次都是这么走运，可能现有的 VLC 都不能满足。

可以使用统计编码解决，这类算法主要是根据符号出现的概率确定唯一的变长编码，这样任意数据集都有自己的一套自定义码字。


### 哈夫曼编码

[Huffman Coding](https://www.programiz.com/dsa/huffman-coding) 其实就是构建一个二叉树

- 每个符号都是叶子节点，所有其编码都满足前缀性质

- 因为是自底向上构建树，且是从权重最小(出现频次最小)的节点开始，那自然出现越频繁的节点路径越短，出现频率低的节点离根越远，整颗树自然也是加权路径最短，也是一颗最优二叉树


Step 1 计算频率

```js
function calFreq(str) {
  const freqTable = {}
  for (let s of str) {
    if (!freqTable[s]) {
      freqTable[s] = 1
    }
    else {
      freqTable[s] += 1
    }
  }
  return freqTable
}
```

Step 2 Build Tree

```js
class Node {
  constructor(char, weight) {
    this.char = char
    this.weight = weight
    this.left = null
    this.right = null
  }
}

function Huffman(str) {
  // building frequency table
  const freqTable = calFreq(str)

  // sorting & creating leaf nodes
  const nodeList = []
  for (let c in freqTable) {
    nodeList.push(new Node(c, freqTable[c]))
  }
  
  // building tree
  while (nodeList.length > 1) {
    nodeList.sort((a, b ) => a.weight - b.weight)
    
    const [l, r] = nodeList.slice(0, 2)
    const newNode = new Node('', l.weight + r.weight)
    newNode.left = l
    newNode.right = r

    nodeList.splice(0, 2, newNode)
  }
  
  return nodeList[0]
    
  // const codings = {}
  // encode(nodeList[0], codings)
}
```

Step 3 encode

```js
function encode(node, coding, c = '') {
  if (!node.left || !node.right) {
    coding[node.char] = c
    return
  }
  
  encode(node.left, coding, c + '0')
  encode(node.right, coding, c + '1')
}
```

来看个实际例子

```js
Huffman('BCAADDDCCACACAC')

A: "11"
B: "100"
C: "0"
D: "101"
```


### 算术编码

这个算法的思想真是好有意思，具体可以参考 [算数编码原理解析](https://segmentfault.com/a/1190000011561822)，写的很详细。


## 字典转换

不同于统计压缩，字典转换是先识别出数据集中常见的单词/长字符串，而不是把每个字母作为一个符号，可以理解为字典转换是预处理阶段。

### 如何找出正确『单词』
- 正确的『单词』：最小熵的字符串
- 怎么识别是单词：分词 `tokenization`

### LZ 算法
把数据中重复出现的长字符串加入字典，后续重复出现的字符串使用标记代替从而进行压缩。算法包括3个概念

- search buffer

- look ahead buffer

- sliding window

具体参考 [LZW压缩算法原理解析](https://segmentfault.com/a/1190000011425787)、[图解 LZ77 压缩算法](https://blog.51cto.com/u_15127629/2873305)，我就懒得再写了。

pros
- 解压很快，比较已经知道位置了

cons
- 压缩比较耗时，需要花时间在 search


## Contextual transform

### run-length encoding

对连续出现相同的符号进行聚类，比如 `AAAABBBBBBBBCCC` 输出为 `(4,A)(8, B),(3,C)`

最坏情况，编码直接翻倍 `ABCD` 输出为 `(1,A)(1,B)(1,C)(1,D)`


### delta coding

将一组数据转换为各个相邻数据之间的相对差值（即增量）的过程。其思想是，给定一组数据，相关的或相似的数据往往会集中在一起。

最适用于处理时间序列数据（比如每10秒检测一次温度的传感器所产生的数据），以及音频和图像数据这类多媒体数据，因为这类数据中邻近的数据之间存在着时间上的关联

比如 `[1,3,6,8,10]` => `[1, 2,3,2,2]`，原先每个字符需要 4 bit，现在只要 2 bit

但是也可能存在负数比如 `[1,3,10,8,6]` => `[1,2,7,-2,-2]`，有很多改进方法

改进一：XOR 增量编码
`[1,3,10,8,6]` => `[1 xor 1, 1 xor 3, 3 xor 10, 10 xor 8, 6 xor 8]` => `[1,2,9,2,14]`

改进二：参照系增量编码
`[107,108,110,115,120,125,132,132,131,135]` 每个数需要 8bit 存储，我们发现这些数都大于 107，所以让每个数都减去 107 就可以进行增量编码，即 `[0,1,3,8,13,18,25,25,24,28]` 这就只需要 5bit


## 评价数据压缩
### 使用场景

- 线下压缩、客户端解压
    - 比如游戏、app 等图片资源，压缩目的是使资源尽可能小

- 客户端压缩、云端解压
    - 减少用户流量费用

- 云端压缩、客户端解压
    - case 1：比如服务器动态生成的数据，压缩目的是使减小网络传输
    - case 2：上传的图片转换为多种格式、大小的图片，上传的视频转为 H.264 等（阿里 oss 等都提供此服务），压缩目的可以高效的将大量数据压缩为最少的二进制位数

- 客户端压缩、客户端解压

### 压缩的需求

- 了解要处理的数据格式、类型、使用方式

- 了解各算法的指标

- 什么场景使用什么样的算法（比如缩略图我们会用有损压缩，而大图模式则使用高清无损图）

### 压缩率

压缩后大小和压缩前大小之比

### 压缩性能

将数据转换为压缩后的形式需要多长时间。在对网络延迟要求很高的情况下，无论是客户端还是服务端负责压，压缩性能都至关重要。考虑 2 个指标
- CPU
- 内存

### 解压性能

压缩率高不一定解压性能好，解压通常在客户端进行，但客户端存在内存小的问题；所以选择压缩算法，也要考虑其解压性能（这也和上面的压缩性能对应，本质还是要综合考虑硬件、CPU、内存等）


## 与前端相关的压缩算法

### 图片压缩

- PNG：无损压缩，支持 alpha 
- JPG：有损压缩，不支持 alpha
- GIF：支持透明、支持动画，仅支持256色
- WEBP：无损+透明度、有损

一般是优先选择 `webp`

### Gzip


### Brotli


## Reference

- [Lucene 4.X 倒排索引原理与实现: (2) 倒排表的格式设计](https://www.cnblogs.com/forfuture1978/p/3944583.html)

- [Huffman Data Compression using JavaScript](https://voskan.host/2023/01/29/huffman-data-compression-using-javascript/)
