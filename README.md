[![npm version](https://badge.fury.io/js/mobx-state-tree.svg)](https://badge.fury.io/js/mobx-state-tree)
[![Build Status](https://travis-ci.org/mobxjs/mobx-state-tree.svg?branch=master)](https://travis-ci.org/mobxjs/mobx-state-tree)
[![Coverage Status](https://coveralls.io/repos/github/mobxjs/mobx-state-tree/badge.svg?branch=master)](https://coveralls.io/github/mobxjs/mobx-state-tree?branch=master)
[![Join the chat at https://gitter.im/mobxjs/mobx-state-tree](https://badges.gitter.im/mobxjs/mobx-state-tree.svg)](https://gitter.im/mobxjs/mobx-state-tree?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

</br>

原文链接：https://github.com/mobxjs/mobx-state-tree

<p>
  <img src="https://raw.githubusercontent.com/mobxjs/mobx-state-tree/master/docs/mobx-state-tree-logo-gradient.svg?sanitize=true" align="left" height="145" width="100"
style="margin-right: 20px"/>
  <h4 style="margin: 0;">mobx-state-tree</h4>
  <p>
    <i>
  Opinionated, transactional, MobX powered state container combining the best features of the immutable and mutable world   for an optimal DX
    </i>
  </p>
</p>

</br>
</br>

> Mobx 和 MST 都是非常令人惊奇的软件, for me it is the missing brick when you build React based apps. Thanks for the great work!# Contents

Nicolas Galle [full post](https://medium.com/@nicolasgall/i-started-to-use-react-last-year-and-i-loved-it-1ce8d53fec6a)
Introduction blog post [The curious case of MobX state tree](https://medium.com/@mweststrate/the-curious-case-of-mobx-state-tree-7b4e22d461f)

---
# Contents

* [安装](#installation)
* [入门指南](docs/getting-started.md)
* [Talks & blogs](#talks--blogs)
* [理念概述](#philosophy--overview)
* [示例](#examples)
* [概念](#concepts)
  * [树、类型和状态](#trees-types-and-state)
  * [创建 model](#creating-models)
  * [树的语义详解](#tree-semantics-in-detail)
  * [树的组成](#composing-trees)
  * [Actions](#actions)
  * [Views](#views)
  * [快照](#snapshots)
  * [补丁](#patches)
  * [引用和标识符](#references-and-identifiers)
  * [Listening to observables, snapshots, patches or actions](#listening-to-observables-snapshots-patches-or-actions)
  * [Volatile state](#volatile-state)
  * [依赖注入](#dependency-injection)
* [Type 概述](#types-overview)
  * [生命周期勾子](#lifecycle-hooks-for-typesmodel)
* [Api 概述](#api-overview)
* [提示](#tips)
* [常见问题](#FAQ)
* [Api 完整文档](API.md)
* [内置的中间件示例](packages/mst-middlewares/README.md)
* [Changelog](changelog.md)

# 安装

* NPM: `npm install mobx mobx-state-tree --save`
* Yarn: `yarn add mobx mobx-state-tree`
* CDN: https://unpkg.com/mobx-state-tree@1.1.0/dist/mobx-state-tree.umd.js (暴露为 `window.mobxStateTree`)
* Playground: [https://mattiamanzati.github.io/mobx-state-tree-playground/](https://mattiamanzati.github.io/mobx-state-tree-playground/) (with React UI, snapshots, patches and actions display)
* CodeSandbox [TodoList demo](https://codesandbox.io/s/nZ26kGMD) fork for testing and bug reporting

Typescript typings are included in the packages. Use `module: "commonjs"` or `moduleResolution: "node"` to make sure they are picked up automatically in any consuming project.

# 入门指南

查看[入门指南](https://github.com/chenxiaochun/mobx-state-tree/blob/master/docs/getting-started.md)教程。

# Talks & blogs

* Talk React Europe 2017: [Next generation state management](https://www.youtube.com/watch?v=rwqwwn_46kA)
* Talk ReactNext 2017: [React, but for Data](https://www.youtube.com/watch?v=xfC_xEA8Z1M&index=6&list=PLMYVq3z1QxSqq6D7jxVdqttOX7H_Brq8Z) ([slides](http://react-next-2017-slides.surge.sh/#1), [demo](https://codesandbox.io/s/8y4p23j32j))
* Talk ReactJSDay Verona 2017: [Immutable or immutable? Both!]() ([slides](https://mweststrate.github.io/reactjsday2017-presentation/index.html#1), [demo](https://github.com/mweststrate/reatjsday2017-demo))
* Talk React Alicante 2017: [Mutable or Immutable? Let's do both!]() ([slides](https://mattiamanzati.github.io/slides-react-alicante-2017/#2))
* Talk ReactiveConf 2016: [Immer-mutable state management](https://www.youtube.com/watch?v=Ql8KUUUOHNc&list=PLa2ZZ09WYepMCRRGCRPhTYuTCat4TiDlX&index=30)

# 理念概述

`mobx-state-tree` is a state container that combines the _simplicity and ease of mutable data_ with the _traceability of immutable data_ and the _reactiveness and performance of observable data_.

Simply put, mobx-state-tree tries to combine the best features of both immutability (transactionality, traceability and composition) and mutability (discoverability, co-location and encapsulation) based approaches to state management; everything to provide the best developer experience possible.
Unlike MobX itself, mobx-state-tree is very opinionated on how data should be structured and updated.
This makes it possible to solve many common problems out of the box.

MST (mobx-state-tree) 的核心思想就是一个动态树。它由严格受保护的易变对象以及运行时类型信息浓缩而成。换句话说，每个树都是由一个结构（类型信息）和一个状态（数据）组成。

通过这个树，共享固定的对象结构，并自动生成快照。

```javascript
import { types, onSnapshot } from "mobx-state-tree"

const Todo = types.model("Todo", {
    title: types.string,
    done: false
}).actions(self => ({
    toggle() {
        self.done = !self.done
    }
}))

const Store = types.model("Store", {
    todos: types.array(Todo)
})

// 通过快照创建一个实例
const store = Store.create({ todos: [{
    title: "Get coffee"
}]})

// 监听新快照
onSnapshot(store, (snapshot) => {
    console.dir(snapshot)
})

// 调用 action 去修改树
store.todos[0].toggle()
// prints: `{ todos: [{ title: "Get coffee", done: true }]}`
```

通过使用这些可用的类型信息，快照可以被转换为一个动态树，反之亦然，也是零操作即可转换回去。因此，MST 默认就能支持[时间旅行](https://github.com/mobxjs/mobx-state-tree/blob/master/packages/mst-example-boxes/src/stores/time.js)和类似于热替换的功能，可参考[示例](https://github.com/mobxjs/mobx-state-tree/blob/4c2b19ec4a6a8d74064e4b8a87c0f8b46e97e621/examples/boxes/src/stores/domain-state.js#L94)

之所以用这样的方式去设计类型信息，就是为了在设计时以及运行时都可以检查类型的正确性（在 TypeScript 中类型检查都是片刻就可以完成的）。

```
[mobx-state-tree] Value '{\"todos\":[{\"turtle\":\"Get tea\"}]}' is not assignable to type: Store, expected an instance of Store or a snapshot like '{ todos: { title: string; done: boolean }[] }' instead.
```

运行时类型错误：

![typescript error](https://github.com/mobxjs/mobx-state-tree/raw/master/docs/tserror.png)

设计时类型错误：

因为状态树是一个动态可变的模型，action 可对它直接进行改写，就像修改本地实例属性一样。可参见上面示例中的`toggleTodo()`方法。不需要你自己手动去产生一个新的状态树，因为 MST 的快照功能会自动产生一个新的出来。

易变的特性听起来好像有些可怕，但是先不要急着害怕，因为 action 给我们提供了很多有趣的特性：
1. 默认情况下，当前树只能被隶属于相同子树的 action 所修改。
2. Action 可以被复制，可以用来分发变化（[example](https://github.com/mobxjs/mobx-state-tree/blob/master/packages/mst-example-boxes/src/stores/socket.js)）。
3. 变化可以在一个相当细的粒度上被检测到，默认支持 [JSON patch](http://jsonpatch.com/)。

Simply subscribing to the patch stream of a tree is another way to sync diffs with, for example, back-end servers or other clients ([example](https://github.com/mobxjs/mobx-state-tree/blob/master/packages/mst-example-boxes/src/stores/socket.js)).

![patches](https://raw.githubusercontent.com/mobxjs/mobx-state-tree/master/docs/patches.png)

MST 无缝结合了 [mobx](https://mobx.js.org) 和 [mobx-react](https://github.com/mobxjs/mobx-react)。更酷的是，因为它默认支持快照、中间件以及可复制的 action，所以它甚至可以替代 Redux 和 reducer。

这甚至可以将 Redux 开发者工具和 MST 联接起来。可查看一个[使用 Redux 与 MST 实现的待办事项示例]。

![开发者工具](https://raw.githubusercontent.com/mobxjs/mobx-state-tree/master/docs/reduxdevtools.png)

MST 内置了引用、标识符、依赖注入、变更记录以及类型循环定义（甚至是跨文件）。

Even fancier: it analyses liveliness of objects, failing early when you try to access accidentally cached information! (More on that later)。

MST 中的一个独特特性就是提供了很灵活的担保，当你试图从对象中读写不属于状态树的部分时，它就会抛出异常。这可以保护你免受那些仍然被引用的对象的意外读取，例如：闭包。

```javascript
const oldTodo = store.todos[0]
store.removeTodo(0)

function logTodo(todo) {
    setTimeout(
        () => console.log(todo.title),
        1000
    )
)

logTodo(store.todos[0])
store.removeTodo(0)
// 使用过期对象，在一秒钟后会抛出异常
```

可查看简洁明了的 [API](API.md) 文档。

---

Another way to look at mobx-state-tree is to consider it, as argued by Daniel Earwicker, to be ["React, but for data"](http://danielearwicker.github.io/json_mobx_Like_React_but_for_Data_Part_2_.html).

类似于 React，MST 是由被称为 model 的组件所构成，它们用来捕捉小块的数据。它们通过 props（快照）被实例化以后，用来管理和保护它们内部的数据（使用 action）。当应用快照的时候，树节点就会尽量保持一致。这就类似于上下文环境机制，可将信息传递给深层的子节点。

An introduction to the philosophy can be watched [here](https://youtu.be/ta8QKmNRXZM?t=21m52s). [Slides](https://immer-mutable-state.surge.sh/). Or, as [markdown](https://github.com/mweststrate/reactive2016-slides/blob/master/slides.md) to read it quickly.

mobx-state-tree "immutable trees" and "graph model" features talk, ["Next Generation State Management"](https://www.youtube.com/watch?v=rwqwwn_46kA) at React Europe 2017. [Slides](http://tree.surge.sh/#1).

# 示例

* [Bookshop](https://github.com/mobxjs/mobx-state-tree/tree/master/packages/mst-example-bookshop) Example webshop application with references, identifiers, routing, testing etc.
* [Boxes](https://github.com/mobxjs/mobx-state-tree/tree/master/packages/mst-example-boxes) Example app where one can draw, drag, and drop boxes. With time-travelling and multi-client synchronization over websockets.
* [Redux TodoMVC](https://github.com/mobxjs/mobx-state-tree/tree/master/packages/mst-example-redux-todomvc) Redux TodoMVC application, except that the reducers are replaced with a MST. Tip: open the Redux devtools; they will work!

# 概念

MobX 状态树，顾名思义，就是用来构建状态的树。

### 树，类型和数据

树中的每一个节点都被两个东西所定义：它的类型（组成结构）和它的数据（目前的状态）。一个最简单的树可能是这样的：

```javascript
import {types} from "mobx-state-tree"

// 定义节点的组成结构
const Todo = types.model({
    title: types.string
})

// 基于 Todo 节点声明创建一个树，并初始化数据
const coffeeTodo = Todo.create({
    title: "Get coffee"
})
```

`types.model`通常用来定义对象的结构。其它内置的 type，还有 array、map和原始类型等，可以查看[types 综述](#types-overview)。

### 创建 model

在 MST 中最重要的类型就是`types.model`，用来描述一个对象的组成结构。例如：

```javascript
const TodoStore = types
    .model("TodoStore", {                             // 1
        loaded: types.boolean                         // 2
        endpoint: "http://localhost",                 // 3
        todos: types.array(Todo),                     // 4
        selectedTodo: types.reference(Todo)           // 5
    })
    .views(self => {
        return {
            get completedTodos() {                    // 6
                return self.todos.filter(t => t.done)
            },
            findTodosByUser(user) {                   // 7
                return self.todos.filter(t => t.assignee === user)
            }
        };
    })
    .actions(self => {
        return {
            addTodo(title) {
                self.todos.push({
                    id: Math.random(),
                    title
                })
            }
        };
    })
```
创建一个 model 时，建议给 model 方法定义一个名称以用于 debug 代码时使用；第二个对象参数定义了它所有的属性。（标有 1 的代码行）

属性参数是一个 key-value 的集合，key 表示属性名称，value 表示它的数据类型。下面的类型都是可接受的：

1. 可以是一个简单的原始类型，例如：`types.boolean`（标有 2 的代码行）；可以是一个复杂的事先定义好的类型（标有 4 的代码行）
2. 可以直接使用一个原始类型值作为默认值（标有 3 的代码行），`endpoint: "http://localhost"`等同于`endpoint: types.optional(types.string, "http://localhost")`。MST 可以通过默认值推测出其数据类型是什么，拥有默认值的属性在创建快照时可以进行省略。
3. 可以是一个[计算属性](https://mobx.js.org/refguide/computed-decorator.html)（标有 6 的代码行）。MobX 会记忆和追踪计算属性。计算属性将不会被存储在快照里，也不会触发 patch 事件。也可能会给计算属性提供一个 setter 方法，并且 setter 方法只能在 action 里被调用。
4. 可以是一个 view 函数（标有 7 的代码行）。View 函数跟计算属性不同，它可以获得任意数量的参数。虽然它不会被记忆，但是它的值可以被 MobX 追踪。View 函数不允许修改 model，通常只是用它来检索 model 的信息。

_提示：`(self) => ({ action1() { }, action2() { }})`是 ES6 的语法，它等价行`function (self) { return { action1: function() { }, action2: function() { } }}`，换句话说，它是一种返回对象字面量的简写方式。
一个 model 的每个成员之间必须强制添加一个逗号，这个与 class 的语法规则是完全不同的。_

`types.model`支持链式写法，每一个链上的方法都会产生一个新的 type：

* `.named(name)`方法会以新的 name 克隆当前 type
* `.props(props)`方法会基于当前产生一个新的 type，并且可以添加或者覆盖掉指定的属性
* `.actions(self => object literal with actions)`方法会基于当前产生一个新的 type，并且可以添加或者覆盖指定的 action
* `.views(self => object literal with view functions)`方法会基于当前产生一个新的 type，并且可以添加或者覆盖指定的 view 方法
* `.preProcessSnapshot(snapshot => snapshot)`通常在实例化一个新的 model 之前用来预处理原始 JSON 数据。可查看[生命周期勾子](#lifecycle-hooks-for-typesmodel)

注意：`views`和`actions`不会直接定义 action 和 view，但仍需要给它们传递一个 function。此 function 会在一个新的 model 实例被创建时引入，此 model 实例会被作为唯一参数传递给 function，通常被命名为`self`。

这会带来两点好处：
1. 所有的方法都会正确的绑定 this 上下文。
2. 闭包通常用来存储实例的私有状态或者方法。可查看[actions](#actions) 和 [volatile state](#volatile-state)。

简单示例：

```javascript
const TodoStore = types
    .model("TodoStore", { /* props */ })
    .actions(self => {
        const instantiationTime = Date.now()

        function addTodo(title) {
            console.log(`Adding Todo ${title} after ${(Date.now() - instantiationTime) / 1000}s.`)
            self.todos.push({
                id: Math.random(),
                title
            })
        }

        return { addTodo }
    })
```

可以很完美的以任意顺序链式调用多个`views`和`props`。这种方式可以用来非常好的组织 types，以及混入一些实用的函数。每次的链式调用都会创建一个可以被它自己存储以及作为其它 types 的一部分重复使用的、新的、不可变的 type。

可以在 action 对象内定义生命周期的勾子，这些拥有预先定义的名称的 action 将会在特定时刻被执行。可查看[生命周期勾子](#lifecycle-hooks-for-typesmodel)。

### 树结构语义详解

MST 树拥有非常特别的语义，这些语义的目的就是为了在你使用 MST 时进行约束。它带来的好处就是提供了各种各样的通用特性，比如：快照、可玩性等。如果这些约束不适用于你的应用，你最好使用普通的 mobx 去结合你的 model 类，这样对于你可能更好一些。

1. 在 MST 中每个对象被认为是一个“node”。每一个原始值被认为是一片“叶子”。
2. MST 仅仅拥有三种节点类型：model、array 和 map。
3. MST 中的每一个 _节点_ 就是它本身。任何可以应用到完整树上的操作都可以应用到子树上。
4. 一个节点只能在一个树中存在一次，这样可确保它是唯一可辨识的。
5. 可以在相同的树中使用 reference 去引用另一个对象。
6. 对存在于一个应用中的 MST 树没有数量限制，但是一个节点仅仅只能存在于一个树中。
7. 树上的所有叶子都必须是可序列化的，否则，就不能被存储，例如：MST 中的函数。
8. The only free-form type in MST is frozen; with the requirement that frozen values are immutable and serializable so that the MST semantics can still be upheld.
9. At any point in the tree it is possible to assign a snapshot to the tree instead of a concrete instance of the expected type. In that case an instance of the correct type, based on the snapshot, will be automatically created for you.
10. Nodes in the MST tree will be reconciled (the exact same instance will be reused) when updating the tree by any means, based on their _identifier_ property. If there is no identifier property, instances won't be reconciled.
11. 如果树中的一个节点被另一个节点替代了，那么源节点就会死去变成不可用的状态。这可以确保你不会在应用程序中意外地操作了过期对象。
12. 如果你想基于树中的已有节点创建一个新的节点，你可以`detach`或者`clone`那个它。

### 树的构成

In MST every node in the tree is a tree in itself.
Trees can be composed by composing their types:

```javascript
const TodoStore = types.model({
    todos: types.array(Todo)
})

const storeInstance = TodoStore.create({
    todos: [{
        title: "Get biscuit"
    }]
})
```

传递给`create`方法的_快照_将会被递归地转换成 MST 节点。因此，你可以安全地这样调用：

```javascript
storeInstance.todos[0].setTitle("Chocolate instead plz")
```

因为树中的任何节点就是它自己本身，所以 MST 中内置的任何方法都可以在任何节点上被调用，而不仅仅是在根节点上。这使得获取某一子树的 patch 流或者是仅仅在某一子树上应用一个中间件都是可行的。

### Actions

默认情况下，只有其中的一个 action 或者更高层级的 action 才能修改节点数据。可以通过传递给`action`的初始化函数中返回一个对象来定义 action。每次实例化时，初始化函数都会被执行，因此，`self`一直指向的都是当前实例。同时，可以在函数中创建用来储存数据的闭包，这种也被称为实例的 _volatile_ 状态；或者是创建只能被 action 调用的私有方法。

```javascript
const Todo = types.model({
        title: types.string
    })
    .actions(self => {
        function setTitle(newTitle) {
            self.title = newTitle
        }

        return {
            setTitle
        }
    })
```

如果没有本地数据和私有方法需要调用的话，也可以简写成：

```javascript
const Todo = types.model({
        title: types.string
    })
    .actions(self => ({ // 注意 `({`，这样返回了一个对象字面量
        setTitle(newTitle) {
            self.title = newTitle
        }
    }))
```

Action 是可被复制的，因此有若干使用约束需要你知道：

- 不使用 action 修改节点将会抛出异常
- 建议 action 的参数都是可序列化的。有些参数是可以被自动序列化，例如，相对于其它节点的相对路径
- 不要在 action 内部使用`this`，应该用`self`来代替它。这使得可以很安全的在没有绑定`this`上下文的函数以及箭头函数中去传递 action

一些有用的方法：

-   [`onAction`](API.md#onaction)列出在 model 中，以及它的后代节点中已经引用的任何 action
-   [`addMiddleware`](API.md#addmiddleware) listens to any action that is invoked on the model or any of its descendants.（此处的官方说明和[API](API.md#addmiddleware)的说明有出入）
-   [`applyAction`](API.md#applyaction)在 model 中，根据给定的 action 描述去引入一个 action

#### 异步的 action

在 MST 中，异步 action 是作为第一级支持的，详情信息描述可查看[这里](docs/async-actions.md#asynchronous-actions-and-middleware)。

异步 action 是用 generator 实现的，并且总是返回一个 promise。可参考实例：[bookshop sources](https://github.com/mobxjs/mobx-state-tree/blob/adba1943af263898678fe148a80d3d2b9f8dbe63/examples/bookshop/src/stores/BookStore.js#L25)。下面是实例的部分主要代码：

```javascript
import { types, flow } from "mobx-state-tree"

someModel.actions(self => {
    const fetchProjects = flow(function* () { // <- note the star, this a generator function!
        self.state = "pending"
        try {
            // ... yield can be used in async/await style
            self.githubProjects = yield fetchGithubProjectsSomehow()
            self.state = "done"
        } catch (error) {
            // ... including try/catch error handling
            console.error("Failed to fetch projects", error)
            self.state = "error"
        }
        // The action will return a promise that resolves to the returned value
        // (or rejects with anything thrown from the action)
        return self.githubProjects.length
    })

    return { fetchProjects }
})
```

#### Action 监听器与中间件的对比

Action 监听器与中间件的区别是：中间件可以主动拦截那些调用了它的 action，modify arguments, return types etc. Action 监听器不能主动拦截，它只能被动的接受通知。Action listeners receive the action arguments in a serializable format, while middleware receives the raw arguments. (`onAction` is actually just a built-in middleware)

对于创建中间件的更多内容可查看[中间件](docs/middleware.md)

#### 禁用保护模式

This may be desired if the default protection of `mobx-state-tree` doesn't fit your use case. For example, if you are not interested in replayable actions, or hate the effort of writing actions to modify any field; `unprotect(tree)` will disable the protected mode of a tree, allowing anyone to directly modify the tree.

### Views

任何从你的数据状态中的派生都可以被称之为“view”或者“derivation”（推导）。想了解更多背景信息可查看[Mobx 概念与原则](https://mobx.js.org/intro/concepts.html)

View 有两种形式：有参数和无参数。后者一般被称为计算值，基于的是 MobX 的[计算](https://mobx.js.org/refguide/computed-decorator.html)概念。两者最主要的区别就是计算属性有一个明确的缓存点，但是它们更深层的工作方式是相同的 and any other computed value or Mobx based reaction like [`@observer`](https://mobx.js.org/refguide/observer-component.html) components can react to them. 计算值是基于 _getter_ 方法定义的。

Example:

```javascript
import { autorun } from "mobx"

const UserStore = types
    .model({
        users: types.array(User)
    })
    .views(self => ({
        get amountOfChildren() {
            return self.users.filter(user => user.age < 18).length
        },
        amountOfPeopleOlderThan(age) {
            return self.users.filter(user => user.age > age).length
        }
    }))

const userStore = UserStore.create(/* */)

// Every time the userStore is updated in a relevant way, log messages will be printed
autorun(() => {
    console.log("There are now ", userStore.amountOfChildren, " children")
})
autorun(() => {
    console.log("There are now ", userStore.amountOfPeopleOlderThan(75), " pretty old people")
})
```

如果你想在 view 和 action 之间共享 volatile 数据，就得使用`.extend`代替`.views`和`.actions`了，可查看[volatile 数据](#volatile-state)。

### 快照

快照就是一个树中特定时间点的、固定的、序列化的纯对象。可以通过使用`getSnapshot(node)`来对它进行检查。

快照不包含任何类型信息并且从所有 action 中剥离了出来，因此，它非常适合用于传输。请求一个快照是非常廉价的，MST 在后台维护着每一个在结构上都是共享的节点快照。

```javascript
coffeeTodo.setTitle("Tea instead plz")

console.dir(getSnapshot(coffeeTodo))
// prints `{ title: "Tea instead plz" }`
```

关于快照的一些有趣的特性：

- 快照是不可变的
- 快照可以用来传输
- 快照可以用来更新 model 或者是恢复它们到某一状态
- 快照可以在需要的时候自动转换成 model。因此下面的两种表述是等价的：`store.todos.push(Todo.create({ title: "test" }))`和`store.todos.push({ title: "test" })`。

一些有用的方法：

- `getSnapshot(model)`：返回一个表示当前 model 状态的快照
- `onSnapshot(model, callback)`：无论何时，当一个新快照可用时，就创建一个监听器(but only one per MobX transaction)
- `applySnapshot(model, snapshot)`：使用快照更新 model 以及它所有后代的状态

## Patches

对一个 model 进行修改时，不仅会产生一个新的快照，而且会生一个描述修改的 [JSON-patches](http://jsonpatch.com/) 流。

Patche 拥有以下特征：

```
    export interface IJsonPatch {
        op: "replace" | "add" | "remove"
        path: string
        value?: any
    }
```
- Patches 是根据 JSON-Patch, RFC 6902 构造的
- Patches are emitted immediately when a mutation is made, and don't respect transaction boundaries (like snapshots)
- Patch 监听器通常用来完成深层次的观察
- 一个 patch 的`path`属性包含着相对于事件监听器被加载的位置的路径
- 一个单一的 mutation 可能会产生多个 patch，例如切分数组的时候
- Patches can be reverse applied, which enables many powerful patterns like undo / redo

一些有用的方法：
- `onPatch(model, listener)`给当前 model 添加一个 patch 监听器，当前 model 或者它的任何后代产生突变时都会被调用
- `applyPatch(model, patch)`给当前 model 应用一个 patch（或者是一个 patch 数组）
- `revertPatch(model, patch)`反向应用一个 patch（或者是一个 patch 数组）到当前提供的 model。这种给 model 反向回放一系列 patch 的方式，通常用来将 model 带回到它的初始的状态

### 引用和标识符

在 MST 中，引用和标识符是一个一级的概念。

This makes it possible to declare references, and keep the data normalized in the background, while you interact with it in a denormalized manner.

Example:
```javascript
const Todo = types.model({
    id: types.identifier(),
    title: types.string
})

const TodoStore = types.model({
    todos: types.array(Todo),
    selectedTodo: types.reference(Todo)
})

// create a store with a normalized snapshot
const storeInstance = TodoStore.create({
    todos: [{
        id: "47",
        title: "Get coffee"
    }],
    selectedTodo: "47"
})

// 因为 `selectedTodo` 被定义为一个标识符，所以它实际返回的是一个与标识符相匹配的 Todo 节点，也就是`id=47`那个节点。console.log(storeInstance.selectedTodo.title)
// prints "Get coffee"
```

#### 标识符

- 每个 model 可定义零个或者一个`identifier()`属性
- 一个对象的标识符属性不可以在后面的初始化中被修改
-   Each identifier / type combination should be unique within the entire tree
-   Identifiers are used to reconcile items inside arrays and maps - wherever possible - when applying snapshots
-   The `map.put()` method can be used to simplify adding objects that have identifiers to [maps](API.md#typesmap)
-   The primary goal of identifiers is not validation, but reconciliation and reference resolving. For this reason identifiers cannot be defined or updated after creation. If you want to check if some value just looks as an identifier, without providing the above semantics; use something like: `types.refinement(types.string, v => v.match(/someregex/))`

_Tip: If you know the format of the identifiers in your application, leverage `types.refinement` to actively check this, for example the following definition enforces that identifiers of `Car` always start with the string `Car_`:_

```javascript
const Car = types.model("Car", {
    id: types.identifier(types.refinement(types.string, identifier => identifier.indexOf("Car_") === 0))
})
```

#### 引用

References are defined by mentioning the type they should resolve to. The targeted type should have exactly one attribute of the type `identifier()`.
References are looked up through the entire tree, but per type. So identifiers need to be unique in the entire tree.

#### 自定义的引用

The default implementation uses the `identifier` cache to resolve references (See [`resolveIdentifier`](API.md#resolveIdentifier)).
However, it is also possible to override the resolve logic, and provide your own custom resolve logic.
This also makes it possible to, for example, trigger a data fetch when trying to resolve the reference ([example](https://github.com/mobxjs/mobx-state-tree/blob/cdb3298a5621c3229b3856bb469327da6deb31ea/packages/mobx-state-tree/test/reference-custom.ts#L150)).

Example:

```javascript
const User = types.model({
    id: types.identifier(),
    name: types.string
})

const UserByNameReference = types.maybe(
    types.reference(User, {
        // given an identifier, find the user
        get(identifier /* string */, parent: any /*Store*/) {
            return parent.users.find(u => u.name === identifier) || null
        },
        // given a user, produce the identifier that should be stored
        set(value /* User */) {
            return value.name
        }
    })
)

const Store = types.model({
    users: types.array(User),
    selection: UserByNameReference
})

const s = Store.create({
    users: [{ id: "1", name: "Michel" }, { id: "2", name: "Mattia" }],
    selection: "Mattia"
})
```

### Listening to observables, snapshots, patches or actions

MST is powered by MobX. This means that it is immediately compatible with `observer` components, or reactions like `autorun`:

```javascript
import { autorun } from "mobx"

autorun(() => {
    console.log(storeInstance.selectedTodo.title)
})
```

But, because MST keeps immutable snapshots in the background, it is also possible to be notified when a new snapshot of the tree is available. This is similar to `.subscribe` on a redux store:

```javascript
onSnapshot(storeInstance, newSnapshot => {
    console.dir("Got new state: ", newSnapshot)
})
```

However, sometimes it is more useful to precisely know what has changed, rather than just receiving a complete new snapshot.
For that, MST supports json-patches out of the box

```javascript
onPatch(storeInstance, patch => {
    console.dir("Got change: ", patch)
})

storeInstance.todos[0].setTitle("Add milk")
// prints:
{
    path: "/todos/0",
    op: "replace",
    value: "Add milk"
}
```

Similarly, you can be notified whenever an action is invoked by using `onAction`

```javascript
onAction(storeInstance, call => {
    console.dir("Action was called: ", call)
})

storeInstance.todos[0].setTitle("Add milk")
// prints:
{
    path: "/todos/0",
    name: "setTitle",
    args: ["Add milk"]
}
```

It is even possible to intercept actions before they are applied by adding middleware using `addMiddleware`:

```javascript
addMiddleware(storeInstance, (call, next) => {
    call.args[0] = call.args[0].replace(/tea/gi, "Coffee")
    return next(call)
})
```

A more extensive middleware example can be found in this [code sandbox](https://codesandbox.io/s/mQrqy8j73).
For more details on creating middleware and the exact specification of middleware events, see the [docs](docs/middleware.md)

Finally, it is not only possible to be notified about snapshots, patches or actions; it is also possible to re-apply them by using `applySnapshot`, `applyPatch` or `applyAction`!

## Volatile state

MST models primarily aid in storing _persistable_ state. State that can be persisted, serialized, transferred, patched, replaced etc.
However, sometimes you need to keep track of temporary, non-persistable state. This is called _volatile_ state in MST. Examples include promises, sockets, DOM elements etc. - state which is needed for local purposes as long as the object is alive.

Volatile state (which is also private) can be introduced by creating variables inside any of the action initializer functions.

Volatile is preserved for the life-time of an object, and not reset when snapshots are applied etc. Note that the life time of an object depends on proper reconciliation, see the [how does reconciliation work?](#how-does-reconciliation-work) section below.

The following is an example of an object with volatile state. Note that volatile state here is used to track a XHR request, and clean up resources when it is disposed. Without volatile state this kind of information would need to be stored in an external WeakMap or something similar.

```javascript
const Store = types.model({
        todos: types.array(Todo),
        state: types.enumeration("State", ["loading", "loaded", "error"])
    })
    .actions(self => {
        const pendingRequest = null // a Promise

        function afterCreate() {
            self.state = "loading"
            pendingRequest = someXhrLib.createRequest("someEndpoint")
        }

        function beforeDestroy() {
            // abort the request, no longer interested
            pendingRequest.abort()
        }

        return {
            afterCreate,
            beforeDestroy
        }
    })
```

Some tips:

1. Note that multiple `actions` calls can be chained. This makes it possible to create multiple closures with their own protected volatile state.
1. Although in the above example the `pendingRequest` could be initialized directly in the action initializer, it is recommended to do this in the `afterCreate` hook, which will only once the entire instance has been set up (there might be many action and property initializers for a single type).

1. The above example doesn't actually use the promise. For how to work with promises / asynchronous flows, see the [asynchronous actions](#asynchronous-actions) section above.

1. It is possible to share volatile state between views and actions by using `extend`. `.extend` works like a combination of `.actions` and `.views` and should return an object with a `actions` and `views` field:

```javascript
const Todo =  types.model({}).extend(self => {
    let localState = 3

    return {
        views: {
            get x() {
                return localState
            }
        },
        actions: {
            setX(value) {
                localState = value
            }
        }
    }
})
```

### model.volatile

In many cases it is useful to have volatile state that is _observable_ (in terms of Mobx observables) and _readable_ from outside the instance.
In that case, in the above example, `localState` could have been declared as `const localState = observable.box(3)`.
Since this is such a common pattern, there is a shorthand to declare such properties, and the example above could be rewritten to:

```javascript
const Todo =  types.model({})
    .volatile(self => ({
        localState: 3
    }))
    .actions(self => ({
        setX(value) {
            self.localState = value
        }
    }))
```

从`volatile`的初始化函数中可以返回任意数量的数据对象，并且它们具有相同名字的实例属性。Volatile 属性具有以下特性：

1. The can be read from outside the model (if you want hidden volatile state, keep the state in your closure as shown in the previous section)
2. The volatile properties will be only observable be [observable _references_](https://mobx.js.org/refguide/modifiers.html). Values assigned to them will be unmodified and not automatically converted to deep observable structures.
3. Like normal properties, they can only be modified through actions
5. Volatile props will not show up in snapshots, and cannot be updated by applying snapshots
5. Volatile props are preserved during the lifecycle of an instance. See also [reconciliation](#reconciliation)
4. Changes in volatile props won't show up in the patch or snapshot stream
4. It is currently not supported to define getters / setters in the object returned by `volatile`

## 依赖注入

When creating a new state tree it is possible to pass in environment specific data by passing an object as the second argument to a `.create` call.
This object should be (shallowly) immutable and can be accessed by any model in the tree by calling `getEnv(self)`.

This is useful to inject environment, or test-specific utilities like a transport layer, loggers etc. This is also very useful to mock behavior in unit tests or provide instantiated utilities to models without requiring singleton modules.
See also the [bookshop example](https://github.com/mobxjs/mobx-state-tree/blob/a4f25de0c88acf0e239acb85e690e91147a8f0f0/examples/bookshop/src/stores/ShopStore.test.js#L9) for inspiration.

```javascript
import { types, getEnv } from "mobx-state-tree"

const Todo = types.model({
        title: ""
    })
    .actions(self => ({
        setTitle(newTitle) {
            // grab injected logger and log
            getEnv(self).logger.log("Changed title to: " + newTitle)
            self.title = newTitle
        }
    }))

const Store = types.model({
    todos: types.array(Todo)
})

// setup logger and inject it when the store is created
const logger = {
    log(msg) {
        console.log(msg)
    }
}

const store = Store.create({
        todos: [{ title: "Grab tea" }]
    }, {
        logger: logger // inject logger to the tree
    }
)

store.todos[0].setTitle("Grab coffee")
// prints: Changed title to: Grab coffee
```

# Types overview

These are the types available in MST. All types can be found in the `types` namespace, e.g. `types.string`. See [Api Docs](API.md) for examples.

## Complex types

* `types.model(properties, actions)` Defines a "class like" type, with properties and actions to operate on the object.
* `types.array(type)` Declares an array of the specified type.
* `types.map(type)` Declares a map of the specified type.

## Primitive types

* `types.string`
* `types.number`
* `types.boolean`
* `types.Date`

## Utility types

* `types.union(dispatcher?, types...)` create a union of multiple types. If the correct type cannot be inferred unambiguously from a snapshot, provide a dispatcher function of the form `(snapshot) => Type`.
* `types.optional(type, defaultValue)` marks an value as being optional (in e.g. a model). If a value is not provided the `defaultValue` will be used instead. If `defaultValue` is a function, it will be evaluated. This can be used to generate, for example, IDs or timestamps upon creation.
* `types.literal(value)` can be used to create a literal type, where the only possible value is specifically that value. This is very powerful in combination with `union`s. E.g. `temperature: types.union(types.literal("hot"), types.literal("cold"))`.
* `types.enumeration(name?, options: string[])` creates an enumeration. This method is a shorthand for a union of string literals.
* `types.refinement(name?, baseType, (snapshot) => boolean)` creates a type that is more specific than the base type, e.g. `types.refinement(types.string, value => value.length > 5)` to create a type of strings that can only be longer then 5.
* `types.maybe(type)` makes a type optional and nullable, shorthand for `types.optional(types.union(type, types.literal(null)), null)`.
* `types.null` the type of `null`
* `types.undefined` the type of `undefined`
* `types.late(() => type)` can be used to create recursive or circular types, or types that are spread over files in such a way that circular dependencies between files would be an issue otherwise.
* `types.frozen` Accepts any kind of serializable value (both primitive and complex), but assumes that the value itself is **immutable** and **serializable**.
* `types.compose(name?, type1...typeX)`, creates a new model type by taking a bunch of existing types and combining them into a new one

## Property types

Property types can only be used as a direct member of a `types.model` type and not further composed (for now).
* `types.identifier(subType?)` Only one such member can exist in a `types.model` and should uniquely identify the object. See [identifiers](#identifiers) for more details. `subType` should be either `types.string` or `types.number`, defaulting to the first if not specified.
* `types.reference(targetType)` creates a property that is a reference to another item of the given `targetType` somewhere in the same tree. See [references](#references) for more details.

## LifeCycle hooks for `types.model`

All of the below hooks can be created by returning an action with the given name, like:

```javascript
const Todo = types
    .model("Todo", { done: true })
    .actions(self => ({
        afterCreate() {
            console.log("Created a new todo!")
        }
    }))
```

The exception to this rule is the `preProcessSnapshot` hook. Because it is needed before instantiating model elements, it needs to be defined on the type itself:

```javascript
types
    .model("Todo", { done: true })
    .preProcessSnapshot(snapshot => ({
        // auto convert strings to booleans as part of preprocessing
        done: snapshot.done === "true" ? true : snapshot.done === "false" ? false : snapshot.done
    }))
    .actions(self => ({
        afterCreate() {
            console.log("Created a new todo!")
        }
    }))
```


| Hook            | Meaning                                                                                                                                                   |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `preProcessSnapshot` | Before creating an instance or applying a snapshot to an existing instance, this hook is called to give the option to transform the snapshot before it is applied. The hook should be a _pure_ function that returns a new snapshot. This can be useful to do some data conversion, enrichment, property renames etc. This hook is not called for individual property updates. _**Note 1: Unlike the other hooks, this one is _not_ created as part of the `actions` initializer, but directly on the type!**_ _**Note 2: The `preProcessSnapshot` transformation must be pure; it should not modify its original input argument!**_ |
| `afterCreate`   | Immediately after an instance is created and initial values are applied. Children will fire this event before parents                                     |
| `afterAttach`   | As soon as the _direct_ parent is assigned (this node is attached to another node). If an element is created as part of a parent, `afterAttach` is also fired. Unlike `afterCreate`, `afterAttach` will fire breadth first. So, in `afterAttach` one can safely make assumptions about the parent, but in `afterCreate` not |
| `postProcessSnapshot` | This hook is called every time a new snapshot is being generated. Typically it is the inverse function of `preProcessSnapshot`. This function should be a pure function that returns a new snapshot.
| `beforeDetach`  | As soon as the node is removed from the _direct_ parent, but only if the node is _not_ destroyed. In other words, when `detach(node)` is used             |
| `beforeDestroy` | Called before the node is destroyed, as a result of calling `destroy`, or by removing or replacing the node from the tree. Child destructors will fire before parents |

Note, except for `preProcessSnapshot`, all hooks should be defined as actions.

All hooks can be defined multiple times and can be composed automatically.



# Api overview

See the [full API docs](API.md) for more details.

| signature | |
| ---- | --- |
| [`addDisposer(node, () => void)`](API.md#adddisposer) | Function to be invoked whenever the target node is to be destroyed |
| [`addMiddleware(node, middleware: (actionDescription, next) => any)`](API.md#addmiddleware) | Attaches middleware to a node. See [middleware](docs/middleware.md). Returns disposer. |
| [`applyAction(node, actionDescription)`](API.md#applyaction) | Replays an action on the targeted node |
| [`applyPatch(node, jsonPatch)`](API.md#applypatch) | Applies a JSON patch, or array of patches, to a node in the tree |
| [`applySnapshot(node, snapshot)`](API.md#applysnapshot) | Updates a node with the given snapshot |
| [`createActionTrackingMiddleware`](API.md#createactiontrackingmiddleware) | Utility to make writing middleware that tracks async actions less cumbersome |
| [`clone(node, keepEnvironment?: true \| false \| newEnvironment)`](API.md#clone) | Creates a full clone of the given node. By default preserves the same environment |
| [`decorate(middleware, function)`](API.md#decorate) | Attaches middleware to a specific action (or flow) |
| [`destroy(node)`](API.md#destroy) | Kills `node`, making it unusable. Removes it from any parent in the process |
| [`detach(node)`](API.md#detach) | Removes `node` from its current parent, and lets it live on as standalone tree |
| [`flow(generator)`](API.md#flow) | creates an asynchronous flow based on a generator function |
| [`getChildType(node, property?)`](API.md#getchildtype) | Returns the declared type of the given `property` of `node`. For arrays and maps `property` can be omitted as they all have the same type |
| [`getEnv(node)`](API.md#getenv) | Returns the environment of `node`, see [environments](#environments) |
| [`getParent(node, depth=1)`](API.md#getparent) | Returns the intermediate parent of the `node`, or a higher one if `depth > 1` |
| [`getPath(node)`](API.md#getpath) | Returns the path of `node` in the tree |
| [`getPathParts(node)`](API.md#getpathparts) | Returns the path of `node` in the tree, unescaped as separate parts |
| [`getRelativePath(base, target)`](API.md#getrelativepath) | Returns the short path, which one could use to walk from node `base` to node `target`, assuming they are in the same tree. Up is represented as `../` |
| [`getRoot(node)`](API.md#getroot) | Returns the root element of the tree containing `node` |
| [`getSnapshot(node)`](API.md#getsnapshot) | Returns the snapshot of the `node`. See [snapshots](#snapshots) |
| [`getType(node)`](API.md#gettype) | Returns the type of `node` |
| [`hasParent(node, depth=1)`](API.md#hasparent) | Returns `true` if `node` has a parent at `depth` |
| [`isAlive(node)`](API.md#isalive) | Returns `true` if `node` is alive |
| [`isStateTreeNode(value)`](API.md#isstatetreenode) | Returns `true` if `value` is a node of a mobx-state-tree |
| [`isProtected(value)`](API.md#isprotected) | Returns `true` if the given node is protected, see [actions](#actions) |
| [`isRoot(node)`](API.md#isroot) | Returns true if `node` has no parents  |
| [`joinJsonPath(parts)`](API.md#joinjsonpath) | Joins and escapes the given path `parts` into a JSON path |
| [`onAction(node, (actionDescription) => void)`](API.md#onaction) | A built-in middleware that calls the provided callback with an action description upon each invocation. Returns disposer |
| [`onPatch(node, (patch) => void)`](API.md#onpatch) | Attach a JSONPatch listener, that is invoked for each change in the tree. Returns disposer |
| [`onSnapshot(node, (snapshot, inverseSnapshot) => void)`](API.md#onsnapshot) | Attach a snapshot listener, that is invoked for each change in the tree. Returns disposer |
| [`process(generator)`](API.md#process) | `DEPRECATED` – replaced with [flow](API.md#flow) |
| [`protect(node)`](API.md#protect) | Protects an unprotected tree against modifications from outside actions |
| [`recordActions(node)`](API.md#recordactions) | Creates a recorder that listens to all actions in `node`. Call `.stop()` on the recorder to stop this, and `.replay(target)` to replay the recorded actions on another tree  |
| [`recordPatches(node)`](API.md#recordpatches) | Creates a recorder that listens to all patches emitted by the node. Call `.stop()` on the recorder to stop this, and `.replay(target)` to replay the recorded patches on another tree |
| [`resolve(node, path)`](API.md#resolve) | Resolves a `path` (json path) relatively to the given `node` |
| [`splitJsonPath(path)`](API.md#splitjsonpath) | Splits and unescapes the given JSON `path` into path parts |
| [`typecheck(type, value)`](API.md#typecheck) | Typechecks a value against a type. Throws on errors. Use this if you need typechecks even in a production build. |
| [`tryResolve(node, path)`](API.md#tryresolve) | Like `resolve`, but just returns `null` if resolving fails at any point in the path |
| [`unprotect(node)`](API.md#unprotect) | Unprotects `node`, making it possible to directly modify any value in the subtree, without actions |
| [`walk(startNode, (node) => void)`](API.md#walk) | Performs a depth-first walk through a tree |
| [`escapeJsonPath(path)`](API.md#escapejsonpath) | escape special characters in an identifier, according to http://tools.ietf.org/html/rfc6901 |
| [`unescapeJsonPath(path)`](API.md#unescapejsonpath) | escape special characters in an identifier, according to http://tools.ietf.org/html/rfc6901 |
| [`resolveIdentifier(type, target, identifier)`](API.md#resolveidentifier) | resolves an identifier of a given type in a model tree |
| [`resolvePath(target, path)`](API.md#resolvepath) | resolves a JSON path, starting at the specified target |

A _disposer_ is a function that cancels the effect it was created for.

# Tips

### Building with production environment

MobX-state-tree provides a lot of dev-only checks. They check the correctness of function calls and perform runtime type-checks over your models. It is recommended to disable them in production builds. To do so, you should use webpack's DefinePlugin to set environment as production and remove them. More information could be found in the [official webpack guides](https://webpack.js.org/plugins/environment-plugin/#usage).

### Generate MST models from JSON

The following service can generate MST models based on JSON: https://transform.now.sh/json-to-mobx-state-tree

### `optionals` and default value functions

`types.optional` can take an optional function parameter which will be invoked each time a default value is needed. This is useful to generate timestamps, identifiers or even complex objects, for example:

`createdDate: types.optional(types.Date, () => new Date())`

### `toJSON()` for debugging

For debugging you might want to use `getSnapshot(model)` to print the state of a model. But if you didn't import `getSnapshot` while debugging in some debugger; don't worry, `model.toJSON()` will produce the same snapshot. (For API consistency, this feature is not part of the typed API)

### Handle circular dependencies between files using `late`

On the exporting file:

```javascript
export function LateStore() {
    return types.model({
        title: types.string
    })
}
```

In the importing file
```javascript
import { LateStore } from "./circular-dep"

const Store = types.late(LateStore)
```

Thanks to function hoisting in combination with `types.late`, this lets you have circular dependencies between types, across files.

### Simulate inheritance by using type composition

There is no notion of inheritance in MST. The recommended approach is to keep references to the original configuration of a model in order to compose it into a new one, for example by using `types.compose` (which combines two types) or producing fresh types using `.props|.views|.actions`. An example of classical inheritance could be expressed using composition as follows:

```javascript
const Square = types
    .model(
        "Square",
        {
            width: types.number
        }
    )
    .views(self => ({
        surface() {
            return self.width * self.width
        }
    }))

// create a new type, based on Square
const Box = Square
    .named("Box")
    .views(self => {
        // save the base implementation of surface
        const superSurface = self.surface

        return {
            // super contrived override example!
            surface() {
                return superSurface() * 1
            },
            volume() {
                return self.surface * self.width
            }
        }
    }))

// no inheritance, but, union types and code reuse
const Shape = types.union(Box, Square)
```

Similarly, compose can be used to simply mixin types:

```javascript
const CreationLogger = types.model().actions(self => ({
    afterCreate() {
        console.log("Instantiated " + getType(self).name)
    }
}))

const BaseSquare = types
    .model({
        width: types.number
    })
    .views(self => ({
        surface() {
            return self.width * self.width
        }
    }))

export const LoggingSquare = types
    .compose(
        // combine a simple square model...
        BaseSquare,
        // ... with the logger type
        CreationLogger
    )
    // ..and give it a nice name
    .named("LoggingSquare")
```

# FAQ

### When not to use MST?

MST makes state management very tangible by offering access to snapshots, patches and by providing interceptable actions.
Also it fixes the `this` problem.
All these features have the downside that they incur a little runtime overhead.
Although in many places the MST core can still be optimized significantly, there will always be a constant overhead.
If you have a performance critical application that handles huge amounts of mutable data, you will probably be better
off by using 'raw' mobx.
Which has predictable and well-known performance characteristics, and has much less overhead.

Likewise, if your application is mainly dealing with stateless information (such as a logging system) MST doesn't add much values.

### How does reconciliation work?

* When applying snapshots, MST will always try to reuse existing object instances for snapshots with the same identifier (see `types.identifier()`).
* If no identifier is specified, but the type of the snapshot is correct, MST will reconcile objects as well if they are stored in a specific model property or under the same map key.
* In arrays, items without an identifier are never reconciled.

If an object is reconciled, the consequence is that localState is preserved and `postCreate` / `attach` life-cycle hooks are not fired because applying a snapshot results just in an existing tree node being updated.

### Creating async flows

See [creating asynchronous flow](docs/async-actions.md).

### Using mobx and mobx-state-tree together

Yep, perfectly fine. No problem. Go on. `observer`, `autorun` etc. will work as expected.

### Should all state of my app be stored in `mobx-state-tree`?
No, or, not necessarily. An application can use both state trees and vanilla MobX observables at the same time.
State trees are primarily designed to store your domain data, as this kind of state is often distributed and not very local.
For local component state, for example, vanilla MobX observables might often be simpler to use.

### Can I use Hot Module Reloading?

Yes, with MST it is pretty straight forward to setup hot reloading for your store definitions, while preserving state. See the [todomvc example](https://github.com/mobxjs/mobx-state-tree/blob/745904101fdaeb51f16f40ebb80cd7fecf742572/packages/mst-example-todomvc/src/index.js#L60-L64)

### TypeScript & MST

TypeScript support is best-effort, as not all patterns can be expressed in TypeScript. But except for assigning snapshots to properties we get pretty close! As MST uses the latest fancy Typescript features it is recommended to use TypeScript 2.3 or higher, with `noImplicitThis` and `strictNullChecks` enabled.

When using models, you write an interface, along with its property types, that will be used to perform type checks at runtime.
What about compile time? You can use TypeScript interfaces to perform those checks, but that would require writing again all the properties and their actions!

Good news! You don't need to write it twice! Using the `typeof` operator of TypeScript over the `.Type` property of a MST Type will result in a valid TypeScript Type!

```typescript
const Todo = types.model({
        title: types.string
    })
    .actions(self => ({
        setTitle(v: string) {
            self.title = v
        }
    }))

type ITodo = typeof Todo.Type // => ITodo is now a valid TypeScript type with { title: string; setTitle: (v: string) => void }
```

Due to the way typeof operator works, when working with big and deep models trees, it might make your IDE/ts server takes a lot of CPU time and freeze vscode (or others)
A partial solution for this is to turn the `.Type` into an interface.
```ts
type ITodoType = typeof Todo.Type;
interface ITodo extends ITodoType {};
```

Sometimes you'll need to take into account where your typings are available and where they aren't. The code below will not compile: TypeScript will complain that `self.upperProp` is not a known property. Computed properties are only available after `.views` is evaluated.

```typescript
const Example = types
  .model('Example', {
    prop: types.string,
  })
  .views(self => ({
    get upperProp(): string {
      return self.prop.toUpperCase();
    },
    get twiceUpperProp(): string {
      return self.upperProp + self.upperProp;
    },
  }));
```

You can circumvent this situation by declaring the views in two steps:

```typescript
const Example = types
  .model('Example', { prop: types.string })
  .views(self => ({
    get upperProp(): string {
      return self.prop.toUpperCase();
    },
  }))
  .views(self => ({
    get twiceUpperProp(): string {
      return self.upperProp + self.upperProp;
    },
  }));
```

Another approach would be to use helper functions, as demonstrated in the following code. This definition allows for circular references, but is more verbose.

```typescript
const Example = types
  .model('Example', { prop: types.string })
  .views(self => {
    function upperProp(): string {
      return self.prop.toUpperCase();
    }
    function twiceUpperProp(): string {
      return upperProp() + upperProp();
    }

    return {
      get upperProp(): string {
        return upperProp();
      },
      get twiceUpperProp(): string {
        return twiceUpperProp();
      },
    };
  });
```

#### Known Typescript Issue 5938

Theres a known issue with typescript and interfaces as described by: https://github.com/Microsoft/TypeScript/issues/5938

This rears its ugly head if you try to define a model such as:

```typescript
import { types } from "mobx-state-tree"

export const Todo = types.model({
    title: types.string
});

export type ITodo = typeof Todo.Type
```

And you have your tsconfig.json settings set to:

```json
{
  "compilerOptions": {
    ...
    "declaration": true,
    "noUnusedLocals": true
    ...
  }
}
```

Then you will get errors such as:

> error TS4023: Exported variable 'Todo' has or is using name 'IModelType' from external module "..." but cannot be named.

Until Microsoft fixes this issue the solution is to re-export IModelType:

```typescript
import { types, IModelType } from "mobx-state-tree"

export type __IModelType = IModelType<any,any>;

export const Todo = types.model({
    title: types.string
});

export type ITodo = typeof Todo.Type
```

It ain't pretty, but it works.

### How does MST compare to Redux

So far this might look a lot like an immutable state tree as found for example in Redux apps, but there are a few differences:

-   Like Redux, and unlike MobX, MST prescribes a very specific state architecture.
-   mobx-state-tree allows direct modification of any value in the tree; it is not necessary to construct a new tree in your actions.
-   mobx-state-tree allows for fine-grained and efficient observation of any point in the state tree.
-   mobx-state-tree generates JSON patches for any modification that is made.
-   mobx-state-tree provides utilities to turn any MST tree into a valid Redux store.
-   Having multiple MSTs in a single application is perfectly fine.

## Contributing

1. Clone this repository
2. Run `yarn run bootstrap` and `yarn run build` once.
3. Extensive pull requests are best discussed in an issue first
3. Have fun!


## Thanks!

* [Mendix](https://mendix.com) for sponsoring and providing the opportunity to work on exploratory projects like MST.
* [Dan Abramov](https://twitter.com/dan_abramov)'s work on [Redux](http://redux.js.org) has strongly influenced the idea of snapshots and transactional actions in MST.
* [Giulio Canti](https://twitter.com/GiulioCanti)'s work on [tcomb](http://github.com/gcanti/tcomb) and type systems in general has strongly influenced the type system of MST.
* All the early adopters encouraging to pursue this whole idea and proving it is something feasible.
