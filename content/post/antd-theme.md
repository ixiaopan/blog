---
title: "Replace antd theme"
date: "2022-05-29"
description: ""
# tags: []
categories: [
  "frontend",
]
series: ["frontend"]
katex: true
---


没有整啥插件，使用 `less variable` 覆盖即可

<!--more-->


## Background

现在大多数中后台开发都是基于第三方UI(如 antd, element-ui)，通常业务也会要求自己的主题定制，这就涉及到所谓的 `换肤`。

庆幸的是，这些 `UI` 提供了这样的功能，拿 `antd` 来说，[themes/default.less](https://github.com/vueComponent/ant-design-vue/blob/98755f332c3c77dea29a0dfcb54615abf4c2ec45/components/style/themes/default.less) 包含了其使用的全部颜色变量，业务只需要覆盖即可。


```css
// -------- Colors -----------
@primary-color: @blue-6;

// >>> Warning
@warning-color: @gold-6;
```


## Why

颜色定制的原理，其实是 `less` 提供了对 `less variables` 的修改，参考 [文档](https://lesscss.org/usage/#using-less-in-the-browser-modify-variables)


```js
less.modifyVars({
  '@buttonFace': '#5B83AD',
  '@buttonText': '#D9EEF2'
});
```

所以只需要在构建里加一个自动 `modifyVars`，注入对应的变量即可

```js
css: {
  preprocessorOptions: {
    less: {
      modifyVars: generateModifyVars(),
      javascriptEnabled: true,
    },
  },
},

function generateModifyVars() {
  return {
    hack: '; @import (reference) ./src/styles/theme.less'
  }
}
```

这是我在网上找到的大多数版本，看到这几行代码，我有几个疑惑

- `javascriptEnabled` 干啥的

- 除了 `modifyVars` 还有其他参数吗

- `hack` 干啥的

- `@import (reference)` 和 `@import` 有什么区别


### Q1 javascriptEnabled

开启这个即允许 `code injection`，但是

> False by default starting in v3.0.0. Enables evaluation of JavaScript inline in .less files. This created a security problem for some developers who didn't expect user input for style sheets to have executable code.

理论上来说，修改 `less variable` 不需要这个参数，但是因为我们依赖的是 `antd` ，这个 `UI` 定义了一个基于 `less` 变量的调色板，这里有一堆 `js function`，比如

```js
.colorPaletteMixin() {
@functions: ~`(function() {
  var getHue = function(hsv, i, isLight) {
  };
}
```

这就导致我们不得不开启这个选项

### Q2 modifyVars

还有 `globalVars` ，区别就是一个是在文件头部，一个在文件最后注入。当然，如果你在文件头部注入，业务恰好有同名变量，也是会被覆盖，这个要注意，所以我们采用 `modifyVars`

```bash
lessc --global-var="color1=red"	{ globalVars: { color1: 'red' } }

lessc --modify-var="color1=red"	{ modifyVars: { color1: 'red' } }
```

除了这两个，我们还可以注入 `banner`，看一下[源码](https://github.com/less/less.js/blob/v4.1.2/packages/less/src/less/parser/parser.js#L159)

```js
parse: function (str, callback, additionalData) {
  let globalVars;
  let modifyVars;
  let preText = '';

  globalVars = (additionalData && additionalData.globalVars) ? `${Parser.serializeVars(additionalData.globalVars)}\n` : '';
  modifyVars = (additionalData && additionalData.modifyVars) ? `\n${Parser.serializeVars(additionalData.modifyVars)}` : '';

  if (globalVars || (additionalData && additionalData.banner)) {
    preText = ((additionalData && additionalData.banner) ? additionalData.banner : '') + globalVars;
  }
}
``` 

### Q3 hack

这个其实你可以用任意的名字，不信看 [源码实现](https://github.com/less/less.js/blob/7491578403a5a35464772c730854c3a5169c0de7/packages/less/src/less/parser/parser.js#L2427)

```js
Parser.serializeVars = vars => {
  let s = '';

  for (const name in vars) {
    if (Object.hasOwnProperty.call(vars, name)) {
      const value = vars[name];
      s += `${((name[0] === '@') ? '' : '@') + name}: ${value}${(String(value).slice(-1) === ';') ? '' : ';'}`;
    }
  }

  return s;
};
```

所以，

```js
{ 
  hack: '; @import (reference) ./src/styles/theme.less;'
}
```

最终会变成 `@hack: ; @import (reference) ./src/styles/theme.less';`


### Q4 @import (reference)

为啥不是 `@import` 呢？？看下 [官方文档](https://lesscss.org/features/#import-atrules-feature-reference) 怎么说的？

> Imagine that reference marks every at-rule and selector with a reference flag in the imported file, imports as normal, but when the CSS is generated, "reference" selectors (as well as any media queries containing only reference selectors) are not output. reference styles will not show up in your generated CSS unless the reference styles are used as mixins or extended.

这种方式可以按需引入，打包的时候，只会编译业务用到的一些变量、类等。


