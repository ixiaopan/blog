---
title: "《Game Programming Patterns》"
date: "2024-01-08"
description: ""
tags: [Reading]
categories: [
  "Frontend",
]
---

最近了读了关于设计模式的一本书，《Game Programming Patterns》虽然是说游戏的，但书中介绍的设计模式在 `web` 开发中也很常见，比如 `singleton/observer/dirty check` 等。不过由于书中代码示例是用 `c++` 编写的，对于前端不是很直观，所以有了从前端角度剖析的想法，权当自身学习。

## Design Patterns

### Observer

> It lets one piece of code announce that something interesting happened without actually caring who receives the notification

既然是观察者模式，那就有被观察的对象和观察者，2个主体

#### Subject/Observer

```ts
class Observer {
  onNotify(sbj, eventName) {}
}
class Subject {
  private observerList: Observer[]
  remove(o: Observer) {}
  add(o: Observer) { this.observerList.push(o) }
  notify(eventName) {
    this.observerList.forEach((o: Observer) => o.onNotify(this, eventName))
  }
}
```

从这段代码可以看到 `subject` 有2个主要工作

1、内部维护了一个数组 `observerList`，表示对 `subject` 发生变化感兴趣的对象即 `observer`；同时支持外部代码添加、删除 `observer`

2、一旦 `subject` 发生变化，就要发送通知 `subject.notify()`

这个模式在前端领域更习惯用 `EventEmitter`，举个图片编辑器的例子

```ts
class ImageObject extends EventEmitter {
  render() {
    this.emit('object:rendered')
  }
}
class Renderer {
  constructor() {
    this.imageObj = new ImageObject()
    this.imageObj.on('object:rendered', () => {
      hideLoading()
    })
  }
  render() {
    showLoading()
    this._objects.forEach((o) => o.render())
  }
}
```

这个例子使用 `EventEmitter` 第三方库，不使用也是可以的，那就要自己包装一个 `EventEmitter` 比如我们对 `resize` 事件感兴趣

```ts
class ResizeEvent {
  private list = []
  constructor() {
    window.addEventListener('resize', (e) => {
      this.list.forEach(fn => fn(e))
    })
  }
  add(fn) {
    this.list.push(fn)
  }
}
```

可见，如果应用中多次使用观察者模式，还是使用 `EventEmitter` 更为方便。

#### Observer VS Pub-Sub

`pub-sub` 即发布订阅模式，和观察者模式很像，区别在于

观察者模式
- 2个主体之间是明确知道对方存在的
- 操作是同步的
- 通信发生在同一个应用内容

`pub-sub`
- 通过 `broker` 联系发布者和订阅者，也就是说可以订阅一个压根不存在的发布者
- 操作可以异步
- 可以跨组件、跨应用通信

同样是对 `resize` 事件感兴趣，使用 `pub-sub` 

```ts
const eventBus = new EventBus()

window.addEventListener('resize', (e) => {
  eventBus.emit('window:resize', e)
})

eventBus.on('window:resize', () => {}) // triggered
eventBus.on('window:scroll', () => {}) // not triggered, because there's no publisher
```


### Singleton

> Ensure a class has one instance, and provide a global point of access to it.

经典的模式，下面的代码大家或多或少都见过

```ts
class ResourceManager {
  static _instance

  constructor() {
    if (!ResourceManager._instance) {
      ResourceManager._instance = new ResourceManager()
    }
    return ResourceManager._instance
  }
}
```

业务中不管在任何地方调用多少次 `new ResourceManager()`，全局都只有一个实例 `ResourceManager._instance`。

这个模式的主要槽点在于实例是全局可访问的，但通常我们都不喜欢变量是全局可访问

- 难以追踪，比如代码中突然出现一个未曾被引入的变量 `xxx` ，你就得查一下这是啥；而且也很难知道哪些地方使用了全局变量

- 容易耦合，比如在游戏中一些东西落在地上要发出声音，很可能就有人在这块业务代码中直接引入 `AudioManager`（显示全局我们只有一个声音管理器），如果每个需要播放声音的地方都这样引入 `AudioManager` 代码就冗余了，后面就麻烦了；更好的方式或许是使用 `observer` 模式通知 `AudioManager` 播放声音


或许下次在考虑单例模式的时候，可以思考一下如何 `limit access scope`

1、从 `base class` 获取

```ts
class Layer {
  log(msg) {}
}
class TextLayer extends Layer {
  doSth() {
    this.log('text layer')
  }
}
```

2、从已存在的全局对象获取

就拿 `AudioManager` 来说，可以挂在 `Editor/Game` 下面，毕竟全局只有一个游戏或者编辑器存在

```ts
class Editor {
  constructor() {
    this.audioManager = new AudioManager()
  }
  playSound(name) {
    // ...
    this.audioManager.play(name)
  }
}

// 业务逻辑：切换不同声音
function changeSound(name) {
  editor.playSound(name)
}
```

3、从 `Service Locator` 获取

定义一个对象 `Service Locator`（这也是一个模式，下文单独介绍），专门提供对全局对象的访问


### State

> States, inputs, transitions
> - You have a fixed set of states that the machine can be in
> - The machine can only be in one state at a time
> - A sequence of inputs or events is sent to the machine
> - Each state has a set of transitions, each associated with an input and pointing to a state.

有限状态机 `finite state machine` 之前在 [迭代管理&发布平台 02](/blog/post/fe/infras-basement-2/) 就有所提及，本质上部署流水线就是一个状态机模型。

我们以书中的例子解释 `FSM`

![](/blog/post/images/fsm.png "Game Programming Pattern")

如图所示有4个状态 standing, jumping, ducking and diving，各状态之间的转换通过用户的输入控制 pressB/pressDown/releaseDown

```ts
enum State {
  STANDING = 'standing',
  JUMPING = 'jumping',
  DUCKING = 'ducking',
  DIVING = 'diving'
}
function handleInput() {
  switch (state) {
    case State.STANDING:
      if (pressB) {
        state = State.JUMPING
        setGraphics(IMAGE_JUMP)
      }
      else if (pressDown) {
        state = State.DUCKING
        setGraphics(IMAGE_DUCK)
      }
      break;
    case State.JUMPING:
      if (pressDown) {
        state = State.DIVING
        setGraphics(IMAGE_DIVE)
      }
      break;
    case State.DUCKING:
      if (releaseDown) {
        state = State.STANDING
        setGraphics(IMAGE_STAND)
      }
      break;
  }
}
```

显然这是我们第一时间想到的写法，实际上我就是这样实现部署流水线的，

```ts
async function start() {
  for (; this.currentStep <= stepList.length - 1; this.currentStep++) {
    const step = stepList[this.currentStep]
    switch (step) {
      case FLOW_STATE.BRANCH_CHECK:
        error = await this.check()
        break
      case FLOW_STATE.BUILD:
        error = await this.build()
        break
      case FLOW_STATE.DEPLOY:
        error = await this.deploy()
        break
      default:
        break
    }
  }
}
```

目前看这样写是没问题，但如果我们想加点东西，比如在按下 `down` 的时候，让游戏角色可以积蓄力量发动某种能力

```ts
let chargeTime
function handleInput() {
  switch (state) {
    case State.STANDING:
      state = State.DUCKING
      setGraphics(IMAGE_DUCK)
      // 新增
      chargeTime = 0
    }
    break;
}
class Game {
  update() {
    // 新增
    if (state == State.DUCKING) {
      chargeTime++
      if (chargeTime > MAX_CHARGE) {
        this.sendBomb()
      }
    }
  }
}
```

可以看到，我们要修改2个地方才能实现这个简单功能，而且还要额外维护一个变量 `chargeTime`，然而这个变量除了和 `ducking` 有关系外，别的地方都不会用到，显然问题原因在于**状态自身逻辑和外部的业务逻辑耦合**。

既如此，那就让每个状态维护自己的逻辑好了，也就是把 `switch/case` 的逻辑放在对应的状态类 `state class` 中，比如 `ducking state`

```ts
class BaseState {
  handleInput() {}
  update() {}
}
class DuckingState extends BaseState {
  private chargeTime = 0
  
  constructor() {
    this.chargeTime = 0
  }
  handleInput() {
    if (releaseDown) {
      // state = State.STANDING 
      game.setGraphics(IMAGE_STAND)
    }
  }
  update() {
    this.chargeTime++
    if (this.chargeTime > MAX_CHARGE) {
      // game 是全局的唯一示例
      game.sendBomb()
    }
  }
}
```

Q：每个 `state` 处理自己的业务逻辑，`state transition` 怎么触发呢，也就是如何实现 `switch/case` 的功能？

A：代理给 `game`，既然同一时间只能有一个状态，那就给全局唯一的游戏实例 `game` 加个指针指向当前的 `state instance`，在 `handleInput` 中实现 `state transition`

```ts
class Game {
  state
  handleInput() {
    this.state.handleInput() // trigger state transition
  }
}
class DuckingState extends BaseState {
  handleInput() {
    if (releaseDown) {
      game.state = StandingState // trigger state transition
      game.setGraphics(IMAGE_STAND)
    }
  }
}
```

Q：`state instance` 从哪里来

A：可以考虑全局就这4个状态

```ts
const GlobalState {
  ducking: new DuckingState(),
  standing: new StandingState(),
}
class Game {
  state = GlobalState.standing
  handleInput() {
    this.state.handleInput() // trigger state transition
  }
}
class DuckingState extends BaseState {
  handleInput() {
    if (releaseDown) {
      game.state = GlobalState.standing // trigger state transition
      game.setGraphics(IMAGE_STAND)
    }
  }
}
```

A：或者每次 `state transition` 生成一个新 `state`，注意这里，我们仅仅是返回了新的 `state`

```ts
class DuckingState extends BaseState {
  handleInput() {
    if (releaseDown) {
      game.setGraphics(IMAGE_STAND)
      return new StandingState() // create a new state instance when we transition to it
    }
  }
}
class StandingState extends BaseState {
  handleInput() {
    if (pressB) {
      game.setGraphics(IMAGE_DUCK)
      return new DuckingState() // create a new state instance when we transition to it
    }
  }
}
class Game {
  state = new StandingState()

  handleInput() {
    const nextState = this.state.handleInput()
    if (nextState) {
      this.state = nextState
    }
  }
}
```

现在代码还有一点小问题，仔细观察就会发现 `game.setGraphics(IMAGE_DUCK)` 的逻辑在 `standingState` 中，应该由 `duckingState` 负责比较合理。换句话说，每个 `state class` 应该自包含所有和它相关的行为和数据

> The goal of the State pattern is to encapsulate all of the behavior and data for one state in a single class.

解决方法也很简单，提供 `state transition` 的 `hooks` 如 `enter/leave` 等，这样就能实现新state的初始化，以及旧state的 `clean up`

```ts
class DuckingState extends BaseState {
  enter() {
    game.setGraphics(IMAGE_DUCK)
  }
  leave() {}
  handleInput() {
    if (releaseDown) {
      return new StandingState()
    }
  }
}
class StandingState extends BaseState {
  enter() {
    game.setGraphics(IMAGE_STAND)
  }
  leave() {}
}
class Game {
  state = new StandingState()

  handleInput() {
    const nextState = this.state.handleInput()
    if (nextState) {
      this.state.leave()
      
      this.state = nextState
      
      this.state.enter()
    }
  }
}
```

### Flyweight

### Command

这个模式在前端不是特别常见，我们先借下游戏开发的一些场景帮助理解 `Command` 是什么。

> A command is a reusable object that represents a thing that can be done. 

很多游戏允许玩家自定义快捷键操作角色，很经典的有些人喜欢用 `up/down/left/right` 控制方向，有些人喜欢用 `wasd`。用代码实现

```ts
handleInput(keyCode) {
  switch (keyCode) {
    case KEY_CODE.UP:
    case KEY_CODE.W:
      jump()
      break
    case KEY_CODE.LEFT:
    case KEY_CODE.A:
      move()
      break
  }
}
```

很明显，这段代码中 `keyCode` 和要执行的动作 `jump/duck/...` 太耦合，无法支持用户自定义快捷命令。解决方法也很显而易见，那就是把 `keyCode` 和执行动作拆开

1、每个动作负责自己的行为

```ts
class Command {
  execute() {}
}
class JumpCommand extends Command {
  execute() { jump() }
}
```

2、绑定 `keyCode` 和动作

```ts
class InputHandler {
  private command = {}
  // 自定义快捷命令
  customCommand(keyCode, cmd) {
    this.command[keyCode] = cmd
  }
  bindDefaultCommand() {
    this.command = {
      [KEY_CODE.UP]: new JumpCommand(),
    }
  }
  handleInput(code) {
    if (isPressed(KEY_CODE.UP)) this.command[KEY_CODE.UP].execute()
  }
}
```

#### Undo/Redo

> Commands can be specific. They represent a thing that can be done at a specific point in time.

`Command` 模式的精华在于，它可以很简单的实现 `undo/redo`。棋牌游戏、编辑软件等都允许用户 `undo/redo`，我之前在做 `PSD` 编辑器就有这个功能，待会再细说业务中怎么使用的，我们先看看 `undo` 是怎么实现的

```ts
class MoveCommand {
  constructor(entity, x, y) {
    this.entity = entity
    this.nextX = x
    this.nextY = y
  }
  execute() {
    this.prevX = this.entity.getX()
    this.prevY = this.entity.getY()
    this.entity.moveTo(this.nextX, this.nextY)
  }
  undo() {
    this.entity.moveTo(this.prevX, this.prevY)
  }
}
```

从这段可以可以看出，`undo` 能实现是因为我们给每个命令拍了快照，执行命令的同时也记住了之前的状态。


#### Record List

在实际的产品中，用户可以 `undo` 多次，然后再次 `redo` 或者执行新的命令，这就要求我们要维护一个 `command list`

- `undo/redo` 实际上是指针的移动
  - `undo`: undo the current command and move the pointer back
  - `redo`: advance the pointer and execute the command
- `undo` 后执行新操作，则是废弃当前指针后的所有命令，把新命令加入到列表
- 另外，列表不可能无限大，一般我们 `undo` 20次就不能再撤销了，所以还需要配置一个可撤销最大次数


```ts
class RecordList {
  private _list: Command[] = []
  private _index: number = -1

  constructor(options) {
    this._max = options.max || 20
  }
  // 可以 重做
  get redoEnabled() {
    return this._index < this._list.length - 1
  }
  // 可以 撤销
  get undoEnabled() {
    return this._index >= 0
  }
  add(cmd: Command) {
    this._list = this._list.slice(0, this._index + 1) // 将栈后面的内容清空
    this._list.push(command)
    this._index++

    // 不能超过最大历史记录
    this._list = this._list.slice(Math.max(0, this._list.length - this._max))
    this._index = this._list.length - 1

    command.exec()
  }
  redo() {
    const cmd = this._list[this._index + 1]
    if (!cmd) return
    this._index += 1
    cmd.execute()
  }
  undo() {
    const cmd = this._list[this._index]
    if (!cmd) return
    this._index -= 1
    cmd.undo()
  }
}
```

以图片编辑器为例，每次修改图层的位置、文字颜色、文字大小等都要存入历史记录

```ts
class Command {
  private _oldV
  private _newV

  private _exec
  private _undo

  constructor(exec, newV: string, oldV: string) {
    this._exec = exec
    this._undo = exec
    this._oldV = oldV
    this._newV = newV
  }

  exec() {
    this._exec(parseJSON(this._newV))
  }

  undo() {
    this._undo(parseJSON(this._oldV))
  }
}

class Editor {
  constructor() {
    this._recorder = new RecordList()
  }
  private setPropsByCommand(d): {
    const ao = this.findObjectByIndex(d.zIndex)
    ao?.set(d)
  }
  addRecord(newV: string, oldV: string) {
    this._recorder?.add(new Command(this.setPropsByCommand.bind(this), newV, oldV))
  }
  undo() {
    this._recorder?.undo()
  }
  redo() {
    this._recorder?.redo()
  }
}
function onModifyLayerAttr(layer, attrName, newVal) {
  const ao = editor?.findObjectByIndex(layer.zIndex)
  
  if (attrName == 'fontColor') {
    editor?.addRecord(
      // 新值
      JSON.stringify({ [attrName]: newVal, zIndex: ao.zIndex }),
      // 旧值
      JSON.stringify({ [attrName]: ao[attrName], zIndex: ao.zIndex })
    )
  }
}
```

- 业务需要修改图层状态，调用 `onModifyLayerAttr` 传入要修改的图层对象、图层属性以及新的属性值

- 这段代码不同于 `MoveCommand` 的地方在于
  - 我们并没有创建 `new FontColorCommand() / new PositionCommand()` 诸如这样的命令，而是把执行命令的方法抽象为 `_exec`，这是因为 `object.set(props, value)` 是修改图层属性的唯一方法
  
  - 要修改的对象也没有直接传入 `new Command`，这里我们是托管到 `editor`，然后通过图层的 `zIndex` 属性找到它（ `setPropsByCommand()` ）

写法虽然有所不同，但本质都是一样，实现 `redo/undo` 的核心都是要保存执行命令那一刻，对象的当下状态。



## Optimization Patterns

### Dirty Flag

> A set of primary data changes over time. A set of derived data is determined from this using some expensive process. A “dirty” flag tracks when the derived data is out of sync with the primary data. It is set when the primary data changes. If the flag is set when the derived data is needed, then it is reprocessed and the flag is cleared. Otherwise, the previous cached derived data is used

用 `vue` 来解释，就是 `computed` 的功能

```ts
const price = ref(0)
const amount = ref(0)
const totalPrice = computed(() => price * amount)
```

## Decoupling Patterns

### Component

### Event Queue

### Service Locator


## Summary

本文所总结的几个模式仅仅是书中的一部分，还有一部分没有提及，主要是因为我在读英文版的时候，总感觉有点晦涩，大概是因为对游戏开发不是很懂以及对 c++ 的不熟悉导致的，所以有一部分内容我就跳过了。但不可否认的是，这本书写的还不错，属于那种常看常新的类型，而且设计模式这种东西本身也比较虚，个人觉得是要有足够的开发经验再去看设计模式才能有更切身的理解。所以跳过的部分待以后再重新拾起来温故知新吧。

## Refer
- [observer vs pub-sub pattern](https://hackernoon.com/observer-vs-pub-sub-pattern-50d3b27f838c)
