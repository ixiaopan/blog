---
title: "Vue3 nextTick/effectScope"
date: "2024-03-22"
description: ""
categories : [
  "Frontend",
]
---

## nextTick

问题：更新响应式数据，并不能立即获取到最新的DOM

```js
<div id="foo">hello, {{ name }}</div>

const name = ref('world')
setTimeout(() => {
  name.value = 'sky'
  console.log('updated', document.querySelector('#foo').innerHTML) // hello, world
}, 2000)
```

解决方法：使用 `nextTick()`

```js
const name = ref('world')
setTimeout(() => {
  name.value = 'sky'
  nextTick(() => {
    console.log('updated', document.querySelector('#foo').innerHTML) // hello, sky
  })
}, 2000)
```

原因：
- 组件更新机制是异步的
- `nextTick` 利用 `promise.then` 保证调用在DOM更新后

### 组件更新

执行 `name.value = 'sky'`，会触发 `side effect`，这里就是更新组件，源码实现如下，可以看到 `compnentUpdateFn` 被包装为 `ReactiveEffect`

```ts
const setupRenderEffect = (instance, initialVNode, container) => {
  const componentUpdateFn = () => {}

  // create reactive effect for rendering
  const effect = (instance.effect = new ReactiveEffect(
    componentUpdateFn,
    NOOP,
    () => queueJob(update), // (**1**)
    instance.scope, // track it in component's effect scope
  ))

  const update = (instance.update = () => {
    if (effect.dirty) {
      effect.run()
    }
  })
    
  update()
}
```

最重要的是注意 `(**1**)` 这行代码，这是个针对 `side effect` 的执行调度，看下源码实现

```ts
const run = (effect) => {
  if (effect.options.scheduler) {
    effect.options.scheduler(effect)
  } else {
    effect()
  }
```


从源码可以看出，一般情况下，响应式数据发生变化，是立刻执行 `componentUpdateFn`，但这里是执行 `() => queueJob(update)`。

如果有如下代码，那每次赋值 `name.value = '' + i` 都会触发 `() => queueJob(update)`，而不是立刻更新组件

```ts
for (let i = 0; i < 10; i++) {
  name.value = '' + i
}
```

那问题来了

-  `() => queueJob(update)` 是干啥的？
- 什么时候更新组件的？

### job queue

```ts
const resolvedPromise = Promise.resolve()
let currentFlushPromise
const queue = []
let isFlushing = false

export function nextTick(this, fn) {
  // 注意这里 用同一个 resolvedPromise
  const p = currentFlushPromise || resolvedPromise
  return fn ? p.then(this ? fn.bind(this) : fn) : p
}

export function queueJob(job) {
  queue.push(job)
  
  if (!isFlushing) {
    isFlushing = true

    // 注意这里 用同一个 resolvedPromise
    currentFlushPromise = resolvedPromise.then(flushJobs)
  }
}

function flushJobs() {
  queue.forEach((f) => f())
}
```

`queueJob(update)` 
- 把 `update` 添加到全局 `queue`
- 如果 `isFlushing = false` 则说明
  - 已经存在 `flushJobs` 的任务
  - 后续的 `queueJobs(job)` 只会把 `job` 添加到全局的 `queue` 中
- 否则，则调用 `resolvedPromise.then(flushJobs)` 开启 `flushJobs` 的任务

`flushJobs()`
- 遍历 `queue` 执行，这里会触发 `componentUpdateFn` 的执行（rerender -> diff -> update dom）


`nextTick(fn)` 
- 再次调用相同的 `resolvedPromise.then` 


当主线程执行完毕，进入 event loop ，开始执行 micro tasks，就会按照 then 的顺序执行，此时 
- `componentUpdateFn` 先执行（rerender -> diff -> update dom），此时 DOM 已经更新，但还未渲染到页面上

- `nextTick(fn)` 中的 `fn` 后执行，此时就获取到了最新的 DOM

- 执行 `ui rendering`

以上就是 `nextTick` 的原理


## forceUpdate

```ts
// componentPublicInstance.ts
$forceUpdate: i =>
  i.f ||
  (i.f = () => {
    i.effect.dirty = true
    queueJob(i.update)
  }),
```


## effectScope

[effectScope - RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0041-reactivity-effect-scope.md) 在 `vue/vuex/pinia` 等多处出现，平时开发中很少用到

> An EffectScope instance can automatically collect effects run within a synchronous function so that these effects can be disposed together at a later time.

如下是官网的例子

```ts
// effect, computed, watch, watchEffect created inside the scope will be collected
const scope = effectScope()

const counter = ref(3)

scope.run(() => {
  const doubled = computed(() => counter.value * 2) // 6 -> 8
  watch(doubled, () => console.log(doubled.value))
  watchEffect(() => console.log('Count: ', doubled.value))
})

counter.value = 4

// to dispose all effects in the scope
scope.stop()
counter.value = 5 // no reactive
```

源码其实很简单，内部就是维护一个 `effect queue`，调用 `stop()` 实际上是调用每个 `effect.stop()` 从从而解除 `track/trigger` 依赖

```ts
let activeEffectScope: EffectScope | undefined

export class EffectScope {
  effects: ReactiveEffect[] = []

  constructor(public detached = false) {}
  
  run(fn) {
    const currentEffectScope = activeEffectScope
    try {
      activeEffectScope = this
      return fn()
    } finally {
      activeEffectScope = currentEffectScope
    }
  }

  stop() {
    for (i = 0, l = this.effects.length; i < l; i++) {
      this.effects[i].stop()
    }
  }
}
```


###  组件 scope
如下是组件实例化的源码实现，每个组件都有对应的 `scope`

```ts
// renderer.ts
const mountComponent = (initialVNode) => {
  const instance = createComponentInstance(initialVNode)
  setupRenderEffect(instance, initialVNode, container)
}
const unmountComponent = (instance) => {
  instance.scope.stop()
}
const setupRenderEffect = (instance, initialVNode, container) => {
  const componentUpdateFn = () => {}

  // create reactive effect for rendering
  const effect = (instance.effect = new ReactiveEffect(
    componentUpdateFn,
    NOOP,
    () => queueJob(update),
    instance.scope, // track it in component's effect scope
  ))
}

// component.ts
export function createComponentInstance(vnode) {
  const instance = {
    vnode,
    type: vnode.type,
    scope: new EffectScope(true /* detached */),
  }
  return instance
}
```

可以发现，上面的代码并没有显示调用 `scope.run()`，这是因为 `new ReactiveEffect` 就已经触发绑定了

```ts
class ReactiveEffect {
  constructor(
    public fn: () => T,
    public trigger: () => void,
    public scheduler?: EffectScheduler,
    scope?: EffectScope,
  ) {
    recordEffectScope(this, scope)
  }
}
function recordEffectScope(effect, scope = activeEffectScope) {
  scope.effects.push(effect)
}
```


### vuex scope

```ts
const scope = effectScope(true)

scope.run(() => {
  forEachValue(wrappedGetters, (fn, key) => {
    // use computed to leverage its lazy-caching mechanism
    // direct inline function use will lead to closure preserving oldState.
    // using partial to return function with only arguments preserved in closure environment.
    computedObj[key] = partial(fn, store)
    computedCache[key] = computed(() => computedObj[key]())
    Object.defineProperty(store.getters, key, {
      get: () => computedCache[key].value,
      enumerable: true // for local getters
    })
  })
})
```

可以看到 `comp` 和 `vuex` 用法有些许不同，不同之处在于 `ReactiveEffect` 是否主动是调用

- 组件实例化，主动调用 `new ReactiveEffect()` 同时传入 `scope` 完成绑定

- vuex 需要借助于 `scope.run(fn)`，因为 `ReactiveEffect()` 是在 `computed()/watch()` 内调用的，这就要解决2个问题
  - 设置 `scope` 为当前 `activeEffectScope`，避免 `new ReactiveEffect(fn, trigger, scheduler?, scope?)` 的 `scope` 参数为空，从而在 `recordEffectScope(effect, scope = activeEffectScope)` 使用 `activeEffectScope` 为默认值
  - 执行 `fn` 触发 `new ReactiveEffect()` 完成绑定
