---
title: "Vite"
date: "2024-03-23"
description: ""
categories : [
  "Frontend",
]
---

`vite` 核心代码分为3大块

- server
- 插件
- 依赖预构建

## server

本文使用 koa 搭建服务器，源码简化如下

```ts
function createServer(options) {
  const config = {
    root: process.cwd(),
    ...options,
    plugins: [
      ...builtInPlugins,
      ...options.plugins,
    ]
  }

  // -- 创建 server 实例
  const app = new Koa()
  
  // -- 插件容器
  const container = createPluginContainer(config)
  app.container = container
  // main transform middleware
  app.use(transformMiddleware())
  // transform index.html
  app.use(indexHtmlMiddleware())

  // -- 依赖预构建
  const listen = app.listen.bind(app)
  app.listen = async (port) => {
    await initDepsOptimizer(config, app)
    listen(port)
  }
  app.listen(3000)
}
```

中间件
- `transformMiddleware` 处理 `js/ts/css/` 等
- `indexHtmlMiddleware` 处理 `html`


插件
- `vite` 内部有很多内置插件，在 `resolveConfig()` 会自动注入，先暂不探究，所以这里用 `builtInPlugins` 直接注入
- `options.plugins` 就是 `vite.config.js` 中的 `plugins` 配置

依赖预构建
- `initDepsOptimizer()` 会在 `server` 启动之前执行 


### indexHtmlMiddleware

`server` 启动之后，打开 `localhost:3000` 会加载 `index.html`，不同的是，这个 `index.html` 头部会注入一行代码，如下所示，主要是和 `hmr` 相关，暂时先不探究，目前我们更关心这是怎么实现的

```ts
<script type="module" src="/@vite/client"></script>
```

可以猜测是上文中的 `indexHtmlMiddleware` 注入了这行代码，源码实际上是调用了 `app.transformIndexHtml()` 再调用 `devHtmlHook()`，通过 `applyHtmlTransforms()` 修改 `html`

```ts
const devHtmlHook: IndexHtmlTransformHook = async () => {
  return {
    html,
    tags: [
      {
        tag: 'script',
        attrs: {
          type: 'module',
          src: path.posix.join(base, CLIENT_PUBLIC_PATH),
        },
        injectTo: 'head-prepend',
      },
    ],
  }
}
```

这里简化如下，通过替换 `</head>` 实现注入 `script`

```ts
function indexHtmlMiddleware() {
  return (ctx) => {
    if (ctx.url == '/') {
      let html =  fs.readFileSync('./index.html', 'utf-8')

      if (__DEV__) {
        // html = app.transformIndexHtml(html)
        // 简单实现
        html = html.replace('</head>', '<script type="module" src="/@vite/client"></script></head>')
      }
      ctx.body = html
      ctx.type = 'html'
    }
  }
}
```

此时 `network` 会有 3 个请求 `index.html, /@vite/client, /src/main.ts`

### transformMiddleware

对于 `/src/main.ts` 会执行另外一个中间件（一个中间件只做一件事）`transformMiddleware`，处理 `js/ts/css/less/vue/images` 等资源，这里我们重点看 `js/css`


```ts
const isJSRequest = (url) =>  /\.([jt]sx?|mjs|vue)/.test(url)
const isCSSRequest = (url) => /\.(css|less|scss)/.test(url)

function transformMiddleware() {
  return async (ctx, next) => {
    if (ctx.method !== 'GET' || ['/', '/favicon.ico'].includes(ctx.url)) {
      return next()
    }

    let url = ctx.url

    // 处理 js/css
    if (isJSRequest(url) || isCSSRequest(url)) {
      const result = await transformRequest(url, app)
      ctx.body = result.code
      ctx.type = 'js'
      // ctx.cacheControl = 'no-cache' / 'max-age=31536000,immutable'
    }

    next()
  }
}
```

- `transformRequest()` 才是真正处理模块的地方
- `ctx.type = 'js'` 设置 `content-type: application/javascript`，让浏览器把 `ts/css` 等都当成 `js` 处理

```ts
async function transformRequest(url, server) {
  let id = await server.container.resolveId(url)

  const res = await server.container.load(id)

  let code = res?.code
  if (!code) {
    code = fs.readFileSync(path.resolve(process.cwd(), id), 'utf-8')
  }

  const ret = await server.container.transform(code, id)
  if (ret) {
    code = ret.code
  }

  return { code, }
}
```

- `server.container` 是通过 `createPluginContainer()` 创建的，其本质是模拟 `rollup` 的插件机制
- 每个模块会经历三个阶段 `resolveId, load, transform`
  - `resolveId(url)` 会把 `import ... from xxx` 中的 `xxx` 解析为磁盘上的绝对路径（`vite:resolve` 就是负责这个的）
  - `load()` 加载其内容，比如 `buffer, text`
  - `transform()` 对原内容进行二次加工，比如注入代码、编译等
- 如果没有一个插件 `load()` 返回结果，读取本地文件

综上，`koa middleware` 中间件负责拦截请求，`container` 负责输出内容

### /@vite/client

`/@vite/client` 比较特殊，是通过 `rollup-alias` 转的

```ts
// config.ts
const clientAlias = [
  {
    find: /^\/?@vite\/env/,
    replacement: path.posix.join(FS_PREFIX, normalizePath(ENV_ENTRY)),
  },
  {
    find: /^\/?@vite\/client/,
    // /@fs/Users/me/todo-app/node_modules/vite/dist/client/env.mjs
    replacement: path.posix.join(FS_PREFIX, normalizePath(CLIENT_ENTRY)),
  },
]
// resolve alias with internal client alias
const resolvedAlias = normalizeAlias(
  mergeAlias(clientAlias, config.resolve?.alias || []),
)
```

在 `rollup-alias` 插件内部会调用自定义的 resolver 也就是 `vite:resolve`

```ts
// explicit fs paths that starts with /@fs/*
if (id.startsWith(FS_PREFIX)) {
  // Users/me/todo-app/node_modules/vite/dist/client/env.mjs
  return id.slice(FS_PREFIX.length)
}
```

## 插件机制

### pluginContainer

`container.resolveId/load/transform()` 实际是遍历 `config.plugins` 调用其对应的方法。

```ts
function createPluginContainer(config) {
  return {
    async resolveId(path, importer) {
      for (let p of config.plugins) {
        if (!p.resolveId) continue

        let id
        try {
          id = await p.resolveId(path, importer)
          break
        } catch (e) {
          console.log('e', e);
        }

        return id
      }
    },
    async load(id) {
      for (let p of config.plugins) {
        if (!p.load) continue

        let ret
        try {
          ret = await p.load(id)
          break
        } catch (e) {}

        return ret
      }
    },
    async transform(code, id) {
      for (let p of config.plugins) {
        if (!p.transform) continue

        let ret
        try {
          ret = await p.transform(code, id)
        } catch (e) {}

        if (!ret) continue
        
        if (typeof ret == 'object') {
          code = ret.code
        } else {
          code = code
        }
      }
      
      return { code }
    }
  }
}
```

现在我们可以写个简单的插件，改写 `src/main.ts` 的内容

```ts
createServer({
  plugins: [
    {
      name: 'vite:mock-main',
      async transform() {
        return { code: 'console.log("hello")' }
      }
    }
  ]
})
```

对于 `css` 的内容，可以通过 `js` 的方式加载

```ts
createServer({
  plugins: [
    {
      name: 'vite:css',
      async transform(code, id) {
        if (isCSSRequest(id)) {
          return { code: `
            const style = document.createElement('style')
            style.setAttribute('type', 'text/css')
            style.innerHTML = \"${code}\"
            document.head.appendChild(style)
          ` }
        }
      }
    }
  ]
})
```

### 插件顺序

插件可以根据 `enforce` 指定运行的顺序，实现很简单

```ts
export function sortUserPlugins(plugins) {
  const prePlugins: Plugin[] = []
  const postPlugins: Plugin[] = []
  const normalPlugins: Plugin[] = []

  if (plugins) {
    plugins.flat().forEach((p) => {
      if (p.enforce === 'pre') prePlugins.push(p)
      else if (p.enforce === 'post') postPlugins.push(p)
      else normalPlugins.push(p)
    })
  }

  return [prePlugins, normalPlugins, postPlugins]
}
```

然后编排到内置插件中就好，可以看到 vite 内置了很多常用的插件，如 `alias/css/json`

```ts
export async function resolvePlugins(config, prePlugins, normalPlugins, postPlugins) {
  return [
    preAliasPlugin(config),
    aliasPlugin({
      entries: config.resolve.alias,
      customResolver: viteAliasCustomResolver,
    }),
    
    ...prePlugins
    
    resolvePlugin(),
    cssPlugin(),
    jsonPlugin(),
    
    ...normalPlugins,
    definePlugin(),
    cssPostPlugin()
    
    ...buildPlugins.pre,
    
    ...postPlugins,
    
    ...buildPlugins.post,
  ].filter(Boolean)
}
```

## 依赖预构建

`vite` 在开发模式，http 服务启动之前，会先进行依赖预构建，下面的代码 `initDepsOptimizer()` 就是用来预构建依赖的

```ts
function createServer(options) {
  // ...
  const listen = app.listen.bind(app)
  app.listen = async (port) => {
    await initDepsOptimizer(config, app) // 依赖预构建
    listen(port)
  }
  app.listen(3000)
}
```

模块的构建分为2大类
- dep 三方依赖，使用 esbuild 提前构建
- 开发者代码，运行时构建

### why
`vite` 基于 `esm` 进行模块的加载，
- dep 可能是非 `esm`，需要进行转换
- 浏览器每次遇到 `import xx` 都会发送请求，出于性能考虑，预构建后只有一个模块需要加载

### .vite

预构建的产物在 `node_modules/.vite/deps` 中，目录结构如下

```ts
- _metadata.json
- vue.js
- vue.js.map
```

可以看到，项目所依赖的 `vue` 被编译为一个文件 `vue.js`，同时还有一个 `_metadata.json` 存储 dep 的元信息

```ts
{
  "hash": "c0bf4fdd",
  "configHash": "31456a99",
  "lockfileHash": "eeb85072",
  "browserHash": "9ffe8d8d",
  "optimized": {
    "vue": {
      "src": "../../vue/dist/vue.runtime.esm-bundler.js",
      "file": "vue.js",
      "fileHash": "ea63625d",
      "needsInterop": false
    }
  },
  "chunks": {}
}
```

- `optimized` 就是预构建的内容，业务代码中的 `import { ref } from 'vue` 会被替换为 `import { ref } from '/node_modules/.vite/deps/vue.js?v=9ffe8d8d`
- `chunks` 是公共依赖，比如另一个库也是依赖 `vue`，`vue` 会被抽取到 `chunks`，暂不探究

### how

先看整体流程，大致分3步

- 扫描依赖
- 保存到 metadata
- 使用 esbuild 构建，输出到 `.vite/deps`

```ts
async function initDepsOptimizer(config, app) {
  depsOptimizer = {
    metadata: {
      optimized: {}
    },
    // getOptimizedDepId: (depInfo) => `${depInfo.file}?v=${depInfo.browserHash}`,
    getOptimizedDepId: (depInfo) => `${depInfo.file}`,
  }

  // 扫描依赖
  // createDepsOptimizer(config, app) -> discoverProjectDependencies(config) -> scanImports(config)
  const deps = await scanImports(config, app)
  console.log('scan deps', deps)

  // 保存到 metadata
  const cacheDir = path.join(config.root, '.vite', 'deps-mock')
  for (let dep in deps) {
    depsOptimizer.metadata.optimized[dep] = {
      src: deps[dep],
      file: path.resolve(cacheDir, dep + '.js'),
    }
  }
  console.log('metadata', depsOptimizer.metadata)

  // 使用 esbuild 构建
  if (!fs.existsSync(cacheDir)) {
    fs.mkdirSync(cacheDir)
  }
  await esbuild.build({
    absWorkingDir: config.root,
    entryPoints: Object.keys(deps),
    bundle: true,
    format: 'esm',
    write: true,
    outdir: cacheDir
  })
}
```

#### scanImport

`vite` 默认以 `**/*.html` 为入口，通过 `esbuild.stdin` 作为 `entry point` 编译，重点是 `esbuildScanPlugin`（注意这个是 esbuild 的插件）

```ts
async function scanImports(config, server) {
  // const entries = glob('**/*.html', {
  //   cwd: config.root,
  //   ignore: [
  //     '**/node_modules/**'
  //   ],
  //   absolute: true,
  // })
  const entries = [path.join(process.cwd(), 'index.html')]
  const deps = {}

  const plugin = esbuildScanPlugin(config, server.container, deps)
  await esbuild.build({
    absWorkingDir: process.cwd(),
    write: false,
    bundle: true,
    format: 'esm',
    stdin: {
      contents: entries.map((e) => `import ${JSON.stringify(e)}`).join('\n'),
      loader: 'js',
    },
    plugins: [plugin],
  })

  return deps
}
```

#### esbuildScanPlugin

我们以下面的代码作为 demo，有5个模块要解析
- `/src/main.ts` 扫描起点
- `vue` 三方依赖，也是预构建的目标
- `./App.vue` 继续扫描
- `./add` 业务代码，继续扫描
- `./main.css` 忽略


```ts
// add.ts
export const add = (a, b) => a + b

// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import './main.css'
import add from './add'

const app = createApp(App)

app.mount('#app')
```

##### html/vue => ts

虽然 `index.html` 是 `entry point`，但是真正的入口是 `<script type=module src="/src/main.ts"></script>`，该如何替换呢？很简单，读取 `html` 找到 `script src="/src/main.ts"` 作为内容返回

- 为什么要使用 `namspace: html`，参考 [issue#3317](https://github.com/evanw/esbuild/issues/3317)
-  `resolveId(path)` 就是上文提到的 `container.resolvedId()`

```ts
function esbuildScanPlugin(config, container, depImports) {
  const seen = new Map()
  const resolveId = async (id, importer) => {
    if (seen.has(id)) {
      return seen.get(id)
    }

    const resolved = await container.resolveId(id, importer)
    seen.set(id, resolved)

    return resolved
  }

  return {
    name: 'vite:dep-scan',
    setup(build) {
      build.onResolve({ filter: /\.(html|vue)$/ }, async ({ path, importer }) => {
        const resolved = await resolveId(path, importer)
        return {
          path: resolved,
          namespace: 'html',
        }
      })

      build.onLoad({ filter: /\.(html|vue)$/, namespace: 'html' }, async ({ path }) => {
        const content = fs.readFileSync(path, 'utf-8')

        const ret = content.match(/<script\s+type="module"\s+src\="(.+)">/)

        if (ret) {
          const js = `import ${JSON.stringify(ret[1])}`
          return {
            contents: js,
            loader: 'js',
          }
        }

        // vue setup 先返回空，源码实现复杂，先不探究
        return {
          loader: 'js',
          contents: 'export default {}',
        }

        // 存储
        // scripts[key] = {
        //   loader,
        //   contents,
        //   resolveDir: normalizePath(path.dirname(p)),
        //   pluginData: {
        //     htmlType: { loader },
        //   },
        // }
        // 通过 虚拟模块导出
        // let virtualPath = ''
        // let js = `export * from ${virtualPath}`
        // return {
        //   contents: js,
        //   loader: 'js',
        // }
      })
    }
  }
}
```

此外，对于 `.vue sfc` 也是以 `html` 的方式处理，不过是通过 virtual module，鉴于这里比较复杂，我们先忽略。

##### css

css 要忽略

```ts
// ignore css
build.onResolve({ filter: /\.(css|less|scss)$/ }, ({ path }) => {
  return {
    path,
    external: true,
  }
})
```

##### bare imports

- `/^[\w@][^:]/` 可以区分 `dep` 和 `asSrc` (也就是业务模块)
- `id.includes('node_modules')` 判断一个模块是不是 `node module`
- `depImports` 是个对象，保存 `dep` 和 `src` 的映射，比如 `{ vue: '/Users/me/todo-app/node_modules/vue/dist/vue.runtime.esm-bundler.js' }`

```ts
const externalEntry = (id) => ({ path: id, external: true })
const isInNodeModules = (id) => id.includes('node_modules')

// bare imports
build.onResolve({ filter: /^[\w@][^:]/ }, async ({ path: id, importer }) => 
{
  if (depImports[id]) {
    return externalEntry(id)
  }

  const resolved = await resolveId(id)

  if (!path.isAbsolute(resolved)) {
    return externalEntry(id)
  }

  if (isInNodeModules(resolved)) {
    depImports[id] = resolved
    return externalEntry(id)
  }
})
```

##### 业务 ts

对于业务模块，通过 `js-loader` 加载对应内容继续扫描依赖

```ts
build.onResolve({ filter: /.*/ }, async ({ path: id, importer }) => {
  const resolved = await resolveId(id, importer)

  if (!path.isAbsolute(resolved)) {
    return {
      path: id,
      external: true,
    }
  }

  return {
    path: resolved,
  }
})

build.onLoad({ filter: /\.(?:j|t)sx?$/ }, async ({ path: id }) => {
  let contents = fs.readFileSync(id, 'utf-8')
  return {
    contents,
    loader: 'js'
  }
})
```


## vite:resolve

上文中，模块的解析都依赖于 `container.resolveId()`，其实际上又是调用 `vite:resolve`，这个方法就是把 `import ... from xxx` 中的`xxx` 解析为磁盘上的绝对路径。简单起见，这里我们考虑3种情况，

- `/` 绝对路径，直接和 cwd 拼起来就行
- `.` 相对路径，要计算 `dirname(importer)`
- `bare imports` 查找 `node_modules/package.json`


```ts
const builtInPlugins = [
  {
    name: 'vite:resolve',
    async resolveId(p, importer) {
      // /src/main.ts => /user/todo-app/src/main.ts
      if (p[0] == '/' && !p.includes(process.cwd())) {
        return path.resolve(process.cwd(), p.slice(1))
      }

      if (p[0] === '.') {
        const basedir = importer ? path.dirname(importer) : process.cwd()
        return path.resolve(basedir, p)
      }

      if (/^[\w@][^:]/.test(p)) {
        // 此模块依赖已构建
        if (depsOptimizer) {
          const depInfo = depsOptimizer.metadata.optimized[p]
          if (depInfo) {
            return depsOptimizer.getOptimizedDepId(depInfo)
          }
        }

        p = tryNodeResolve(p, importer)
      }

      return p
    }
  },
]

function tryNodeResolve(id, importer) {
  let baseDir = process.cwd()

  // resolvePackageData()
  const pkgDir = path.join(baseDir, 'node_modules', id)
  const pkgData = JSON.parse(fs.readFileSync(path.join(pkgDir, 'package.json'), 'utf-8'))

  // resolvePackageEntry()
  let entryPoint
  if (pkgData.exports) {}
  if (!entryPoint) {
    // ['browser', 'module', 'jsnext:main', 'jsnext]
    entryPoint = pkgData.module
  }

  entryPoint ||= pkgData.main

  const entryPath = path.join(pkgDir, entryPoint)

  console.log('tryNodeResolve done: ', entryPath)
  return entryPath
}
```


## vite:import-analysis

最后一个问题，`import xxx from vue` 是如何替换为 `import {createApp} from '/.vite/deps-mock/vue.js'`

上文中，`deps = await scanImports()` 已经保存在了 `depsOptimizer.metadata`，其数据结构如下


```ts
metadata {
  optimized: {
    vue: {
      src: '/Users/me/todo-app/node_modules/vue/dist/vue.runtime.esm-bundler.js',
      file: '/Users/me/todo-app/.vite/deps-mock/vue.js'
    }
  }
}
```

可以猜测，是在 transform 阶段替换了 `from vue` 为 `depsOptimizer.metadata.vue.file`，具体实现如下

```ts
{
    name: 'vite:import-analysis',
    async transform(source, importer) {
      // [imports, exports] = parseImports(source)
      source = source.replace(/[\r\n]/g, ';')

      const res = source.match(/import[\s{}\w]+from\s+'([\w\/\.]+)'/g)
      if (res) {
        // [import { createApp } from 'vue', import App from './App.vue']
        const imports = res.reduce((memo, im) => {
          const [_, n] = im.match(/from\s+'([\w\.\/]+)'/)
          const s = source.indexOf(n)
          memo.push({
            n,
            s,
            e: s + n.length
          })
          return memo
        }, [])

        console.log('imports', imports);

        imports.forEach((im) => {
          // 实际调用 this.resolveId()，这里简单模拟
          let url = im.n
          if (/^[\w@][^:]/.test(im.n)) {
            // 此模块依赖已构建
            if (depsOptimizer) {
              const depInfo = depsOptimizer.metadata.optimized[im.n]
              if (depInfo) {
                url = depsOptimizer.getOptimizedDepId(depInfo)
              }
            }
            else {
              url = tryNodeResolve(im.n)
            }


            url = url.slice(process.cwd().length)

            console.log('rewritten url', url)

            // TODO: 忽略 offset，源码使用 magic string
            source = source.slice(0, im.s) + url + source.slice(im.e)
          }
        })

        return { code: source }
      }
    }
  }
```

## 优化手段

- 开启图片压缩 vite-plugin-imagemin

- tree-shaking

- rollup-plugin-visualizer 分析模块大小

- 按需导入组件

  ```ts
  Components({
    resolvers: [
      AntDesignVueResolver({
        importStyle: 'less',
      }),

      (name) => {
        if (name.startsWith('M')) {
          const dirName = kebabCase(name.slice(1))

          return {
            importName: name,
            path: '@lib/lib-ui/es',
            sideEffects: `@lib/lib-ui/es/components/${dirName}/style/index`,
          }
        }
      },
    ],
  }),
  ```

- 资源分类

  ```ts
  build: {
    rollupOptions: {
      output: {
        chunkFileNames: 'js/[name]-[hash].js', // 引入文件名的名称
        entryFileNames: 'js/[name]-[hash].js', // 包的入口文件名称
        assetFileNames: '[ext]/[name]-[hash].[ext]', // 资源文件像 字体，图片等
      }
    }
  }
  ```

- 拆分包

  ```ts
    build: {
      rollupOptions: {
        output: {
          manualChunks(id) {
            if (id.includes('node_modules')) {
              return 'vendor' // 合并为一个文件
              // return id.split('node_modules')[1].split('/')[0] // 分开打包
            }
          }
        }
      }
    }
  ```

- terser && drop console

  ```ts
  build: {
    minify: isDev ? false : 'terser',
    target: 'es2015',
    terserOptions: isDev
      ? undefined
      : {
          compress: {
            keep_infinity: true,
            drop_console: VITE_DROP_CONSOLE,
            drop_debugger: VITE_DROP_CONSOLE,
          },
        },
  }
  ```

## See also
- https://juejin.cn/post/7103730522517569567