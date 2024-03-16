---
title: "Modules"
date: "2024-03-15"
description: ""
categories : [
  "Frontend",
]
---

## Module

模块的作用
- 类似命名空间的上下文隔离，避免各个功能模块间的变量冲突
- 支持模块间的引用即依赖，避免把需要用到的变量、方法等提升到全局作用域

主流方案
- Commonjs，适用于 Node
- AMD，浏览器实现，代表 requireJS
- ES Module，ECMA Spec

```ts
// commonjs
const fs = require('fs')

// esm
import fs from 'js'
```

本文只讨论 `commonjs/esm`，原因如下
- node 社区很多库还是以 commonjs 为主
- esm 是 spec，大势所趋

## Commonjs

- 每个文件都是一个模块
- `module.exports / exports` 导出
- `require` 导入依赖
- 运行时加载


### module.exports/exports

1、真正的导出是 `module.exports`，`exports` 只是对其的引用


```ts
// c.js
exports.color = 'red'
module.exports = { color: 'blue' }

// index.js
const mod = require('./c')
console.log('mod.color:', mod); // { color: 'blue' }
```

2、导出其实就是赋值，所以不管是 primitive 还是 object 是 copy value，只是前者是 copy 变量值，后者是 copy 指向该对象的引用

另外，多次 `require` 只会执行一次

```ts
// c.js
let color = 'red'
let person = { name: 'alice' }

exports.color = color
exports.person = person

console.log('run c', this) // this 指向当前模块

setTimeout(() => {
  color = 'blue'
  // exports.color = color // this line will export 'blue' color
  person.name = 'bob'
}, 1000)

// index.js
const mod = require('./c')
console.log('mod.color:', mod)

setTimeout(() => {
  const mod = require('./c') // 再次 require
  console.log('mod.color after 2s:', mod);
}, 2000);

// node index.js
// run c { color: 'red', person: { name: 'alice' } }
// mod.color: { color: 'red', person: { name: 'alice' } }
// mod.color after 2s: { color: 'red', person: { name: 'bob' } }
```

### 模块加载原理

node 源码看这里 [lib/internal/cjs/loader.js](https://github.com/nodejs/node/blob/main/lib/internal/modules/cjs/loader.js)，这里给出一个简单实现，用来解释前面的例子

```ts
class Module {
  constructor(id) {
    this.id = id
    this.exports = {}
    this.loaded = false
  }

  static _cache = Object.create(null)
  
  static _resolveFilename(id, parent, isMain, options) {
    let paths
    const filename = Module._findPath(id, paths, isMain)
    if (filename) return filename
  }
  
  static _extensions = {
    ['.js'](module) {
      const script = fs.readFileSync(module.id, 'utf-8')
      
      const exports = module.exports
      const require = myRequire
      const thisValue = exports
      const filename = module.id
      const dirname = path.dirname(filename)
      
      // 源码
      // const compiledWrapper = wrapSafe(filename, script, module)
      // ReflectApply(compiledWrapper, thisValue, [exports, require, module, filename, dirname]);
      
      // 等同于
      const fn = `(function(exports, require, module, __filename, __dirname) { ${script} })`
      fn.call(thisValue, exports, require, module, filename, dirname)
    },
    ['.json'](module) {
      const content = fs.readFileSync(module.id, 'utf-8')
      module.exports = JSON.parse(content)
    }
  }

  load(filename) {
    let ext = path.extname(filename)
    
    Module._extensions[ext](this)
    
    this.loaded = true
  }
}

function myRequire(id) {
  const absPath = Module._resolveFilename(id)
  
  const cached = Module._cache[absPath]
  if (cached) {
    return cached.exports
  }

  const mod = new Module(id)
  Module._cache[id] = mod
  
  mod.load(absPath)
  return mod.exports
}
```


### 文件查找

Files modules
- rule 1: 查找同名文件，没有找到，添加 `.js/.json/.node` 继续找
- 找不到，报错 `MODULE_NOT_FOUND`

想要加载 `.cjs`，必须写成这样 `require('./xx.cjs')`


Folders as modules
- rule 2: 查找最近带有 `package.json` 同名目录，根据 `pkg.main` 加载
- rule 3: 没有 `package.json`，查找同名目录下的 `index.js`
- 找不到，报错 `Cannot find module xxx`


比如加载 `require('./my-lib')`，其 `package.json` 如下

```ts
{
  "main": "./src/gulp.js"
}
```

- 尝试读取 `package.json` 加载 `./my-lib/src/gulp.js`
- 如果没有 `package.json` 或者 `main` 不存在，尝试加载 `./my-lib/index.js, ./my-lib/index.node`



```ts
Module._findPath = function(request, paths, isMain) {
  const basePath = path.resolve(request)
  
  const rc = _stat(basePath)
  let filename
  
  if (!trailingSlash) {
    // rule 1
    if (rc === 0) { // file
      filename = path.resolve(basePath)
    }

    if (!filename) {
      // Try it with each of the extensions
      if (exts === undefined) {
        exts = ObjectKeys(Module._extensions);
      }
      filename = tryExtensions(basePath, exts, isMain);
    }
  }

  // dictionary
  if (!filename && rc === 1) {
    // try it with each of the extensions at "index"
    if (exts === undefined) {
      exts = ObjectKeys(Module._extensions);
    }
    filename = tryPackage(basePath, exts, isMain, request);
  }
}

function tryPackage() {
  const { main: pkg, pjsonPath } = _readPackage(requestPath);

  // rule 3
  if (!pkg) {
    return tryExtensions(path.resolve(requestPath, 'index'), exts, isMain)
  }

  // rule 2
  const filename = path.resolve(requestPath, pkg);
  let actual = tryFile(filename, isMain) ||
    tryExtensions(filename, exts, isMain) ||
    tryExtensions(path.resolve(filename, 'index'), exts, isMain);
}

function tryExtensions(basePath, exts: string[], isMain) {
  for (let i = 0; i < exts.length; i++) {
    const filename = tryFile(basePath + exts[i], isMain);

    if (filename) {
      return filename;
    }
  }
  return false;
}
```

### 循环引用

参考 node 官网的例子

```ts
// a.js
console.log('a starting');
exports.done = false;
const b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done'); 

// b.js
console.log('b starting');
exports.done = false;
const a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done'); 

// main.js
console.log('main starting');
const a = require('./a.js');
const b = require('./b.js');
console.log('in main, a.done = %j, b.done = %j', a.done, b.done); 
```

简单理解为，其实普通 js 函数的执行，因为每个模块都被 node 转为了匿名函数内的作用域下

```ts
(function(exports, require, module, __filename, __dirname) {
  // Module code actually lives in here
}); 
```

- 初始化模块的时候，`exports` 就已经存储在 `Module._cache[id] = module` 了，后续再次引入相同的模块都只是从缓存中获取 `exports`

- 运行时加载
  - 代码运行时，才会给 `exports` 赋值，所以是动态绑定
  - 遇到 `require()` 会新增一个 call stack，导致前一个调用父上下文暂时 pause，所以此时 exports 存储的是当前运行阶段的值，而不是代码跑完最终的值；
  - 只有回到父上下文，才会继续执行父上下文中的代码，即 `exports.xxx`；
  - 如果调用的 `require()` 的时机比 `exports.xxx` 要早，这种情况很容易得到 undefined，如下方代码所示

- 所以 `require` 的顺序会影响执行的结果

```ts
// a.js
require('./b')
exports.person = { age: 20 }

// b.js
const mod = require('./a')
console.log(mode.person) // undefined
exports.name = 'hello'
```


## ES Module

### export/import

```ts
// -- export before declarations
export const name = 'bob'
export class User {}

// -- standalone export
let age = 21
class Animal {}
export { Animal, age }

// -- alias
export { age as price, User as Person }

// -- re-export: import things and immediatelly export them
// syntax: `export ... from ...` 
export { age } from './util'
export { age as price } from './util'
export * from './util' // re-export named exports only
export { default as User } from './util'
export { default } from './util'

// -- default export
// - a library that contains a bunch of functions, e.g. `util.js`
// - declare a single entity, e.g. `class User`
// case 1
export default class User {}

// case 2 匿名默认导出
export default class {}
export default function(a, b) { return a + b}
export default ['Jan', 'Feb']

// case 3 use 'default' as a reference to the default export
function add() {}
export { add as default }

// -- import named exports
import { add, User } from './util'
import { add as sum } from './util'
import { myAdd, MyUser } from './util' // wrong, won't work

// -- import everything
import * as util from './util' 

// -- import the default
// case 1
import User from './util'
import MyUser from './util' // it works, since variable name can be anything

// case 2 use 'default' as a reference to the default export
import { default as x, add } from './util'

// case 3 use 'default' as a reference to the default export
import * as user from './user'
const User = user.default
user.add()
```

### 加载原理

[ES modules: A cartoon deep-dive on hacks.mozilla.org (2018)](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)


## ESM Support

### 打包工具


### 浏览器

原生支持，通过 `<script type="module">` 加载 esm


### Node

node 通过2种方式支持 commonjs 和 esm 
- 指定后缀名
- 根据 package.json 中的 type 区分

具体来说
- `.mjs`: ems
- `.cjs`: commonjs
- `.js`:
  - type: 'commonjs' 或者 没有 type => commonjs，最好把 esm 命名为 `xxx.mjs`
  - type: 'module' => esm，如果需要 commonjs，需要显示命名为 `xxx.cjs`

此外，`esm` 可以引用 `commonjs`，但是 `commonjs` 无法引入 `esm`，具体参考 [packages - Node Doc](https://nodejs.org/api/packages.html#determining-module-system)


### pkg.exports

现在，如何写才能让一个库既支持 commonjs 又支持 esm 呢？答案是 `exports`

```ts
{
  "exports": {
    "import": "./xx.js",
    "require": "./xx.cjs"
  },
  "type": "module",
  "main": "./xxx.cjs"
}
```

### 双包问题

但是，既支持 commonjs 又支持 esm ，会导致代码被执行2次，有2个方法吧
- 代码用 commonjs 写，用 esm 包装 commonjs
- 只支持 esm


## See also
- [Modules - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [ES modules: A cartoon deep-dive on hacks.mozilla.org (2018)](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)
- [Module Cycles - Node Doc](https://nodejs.org/api/modules.html#modules_cycles)
- [dual-commonjses-module-packages - Node Doc](https://nodejs.org/api/packages.html#dual-commonjses-module-packages)
- [Modules - Javascript info](https://javascript.info/modules-intro)
- [聊聊ESModule - 掘金](https://juejin.cn/post/7132061292801556517)
- [通俗讲解JS模块的循环引用问题 - 掘金](https://juejin.cn/post/7085029980899377160)
