# 「翻译」8 步创建你自己的 React
- 源地址：https://pomb.us/build-your-own-react/
- repo: https://github.com/pomber/didact
- sandbox:https://codesandbox.io/s/didact-8-21ost

## 第 0 步，回顾

首先们回顾一些基本概念，如果你已经清楚 React, JSX 和 DOM 是如何工作的，你可以跳过这一节

我们用这三行代码构建了一个 React 应用。第一行定义了一个 React 元素，第一行获得了一个 DOM 元素，最后一行讲 React 元素渲染到这个容器中。

```jsx
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

**让我们将其中的 React 特定的代码换成普通的 JS 代码**

第一行我们用 JSX 定义了一个元素。它不是有效的 JS 代码，所以我们打算替换它。

JSX 可以使用类似 Babel 一类的工具转换成 JS 代码。转换规则十分简单：调用 `createElement` 方法来替换标签里的代码，并传入标签名、子元素以及各种参数。

`React.createElement` 根据传入的参数创建了一个对象。抛开一些验证的话，这就是全部的转换过程，我们可以安全地将这个函数的输出替换过来。

```javascript
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
)
​
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

这就是元素了，一个拥有两个参数的对象：`type` 和 `props` （其实还有更多，但我们只关心这两个）

```javascript
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
```

`type` 属性是一个字符串，指定了我们想要创建的 DOM 节点的类型，它是你想要创建 HTML 元素时，传给 `document.createElement` 方法的 `tagName` 参数。我们会在第 7 步的时候将它变为一个函数。

`props` 是另外一个对象，它拥有来自 JSX 属性里的所有键值对。它还有个特殊的参数: `children`

在这个例子里，`children` 是一个字符串，但它通常是一个许多元素的数组。这页是为什么元素通常用树形表示。

另外一个需要替换的 React 代码是 `ReactDOM.render` 调用。

```jsx
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
​
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

`render` 是 React 用来改变 DOM 的方法，所以让我们自己来更新这些。

首先我们用元素的 `type` 来创建一个节点，在这个例子里是 `h1`

接着们将元素所有的 `props` 分配给这个节点，在这里只有一个标题。

> ***为了避免混淆，我用 "element" 来表示 React 元素，用 "node" 来表示 DOM 元素***

```javascript
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
​
const container = document.getElementById("root")
​
const node = document.createElement(element.type)
node["title"] = element.props.title
```

接下来我们 `children` 节点。我们只有一个字符串作为子节点，所以我们创建一个文本节点。

使用 `textNode` 来替代 `innerText` 可以是与我们使用同样的方式来对待所有的元素。注意我们像设置 `h1` 的`title` 一般来设置 `nodeValue`，如同这个字符串拥有属性 `props: {nodeValue: "hello"}`

```javascript
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
​
const container = document.getElementById("root")
​
const node = document.createElement(element.type)
node["title"] = element.props.title
​
const text = document.createTextNode("")
text["nodeValue"] = element.props.children
```

最后，我们将 `textNode` 节点加在 `h1` 上，将 `h1` 加在 `container` 上。

这样我们就成功替代了代码，不再使用 React 本身了。

```javascript
const element = {
  type: "h1",
  props: {
    title: "foo",
    children: "Hello",
  },
}
const container = document.getElementById("root")
​
const node = document.createElement(element.type)
node["title"] = element.props.title
​
const text = document.createTextNode("")
text["nodeValue"] = element.props.children
​
node.appendChild(text)
container.appendChild(node)
```

## 第 1 步，`createElement` 函数

让我们从另外一个应用开始。这次我们会用自己创造的 React 来替换原来的 React 代码。

我们从写自己的 `createElement` 函数开始。

让我们将 JSX 转变成 JS，这样我们可以看到 `createElements` 的调用。

```jsx
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

在之前我们看到，元素是一个拥有 `type` 和 `props` 属性的对象。我们的函数要做的一件事就是创建这个对象。

```javascript
const element = React.createElement(
  "div",
  { id: "foo" },
  React.createElement("a", null, "bar"),
  React.createElement("b")
)
const container = document.getElementById("root")
ReactDOM.render(element, container)
```

我们为 `props` 指定了参数名，而使用***剩余参数***的语法来传入 `children` 这样 `children` 参数就会是一个数组了。

```javascript
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children,
    },
  }
}
```

举个例子, `createElment("div")` 将会返回：

```javascript
{
  "type": "div",
  "props": { "children": [] }
}
```

`createElement("div", null, a)` 返回：

```javascript
{
  "type": "div",
  "props": { "children": [a] }
}
```

以及 `createElement("div", null, a, b)` 返回：

```javascript
{
  "type": "div",
  "props": { "children": [a, b] }
}
```

`children` 数组同样也可以拥有原始类型的值，比如字符串或数字。所以为了同样可以包装在元素中不是对象的情况，我们需要创建一个特别的类型 `TEXT_ELEMENT`。

在没有 `children` 的时候，React 是不会包装原始类型的值或者创建空数组的，但我们这样做的话可以简化代码，因为对于这个例子，我们需要代码更简单而不是效率更高。

```javascript
function createElement(type, props, ...children) {
  return {
    type,   
	props: {
      ...props,
      children: children.map(child =>
        typeof child === "object"
          ? child
          : createTextElement(child)
      ),
    },
  }
}
​
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
```

我们仍然在使用 React 的 `createElement` 

为了替换它，我们需要给我们的库取个名字。这样就可以像 React 一样，并且提示我们是为了教学的目的。

我们叫它 Didact

而我们仍然想使用 JSX。我们怎样才能让 babel 使用 Didact 的 `createElement` 来代替 React 的？

```javascript
const Didact = {
  createElement,
}
​
const element = Didact.createElement(
  "div",
  { id: "foo" },
  Didact.createElement("a", null, "bar"),
  Didact.createElement("b")
)
```

如果我们添加这样的注释，babel 编译 JSX 的时候就会用我们定义的函数了

```jsx
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
```

## 第 2 步，`render` 函数

接下来，我们需要写自己版本的 `ReactDOM.render` 函数。

```javascript
ReactDOM.render(element, container)
```

到目前为止，我们只关注如何往 DOM 中添加内容。我们会在之后来进行更新和删除操作。

```jsx
function render(element, container) {
  // TODO create dom nodes
}
​
const Didact = {
  createElement,
  render,
}
​
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
Didact.render(element, container)

```

我们从使用一个元素的 `type` 来创建一个 DOM 节点开始，然后再这个容器里添加新的节点

```javascript
function render(element, container) {
  const dom = document.createElement(element.type)
​
  container.appendChild(dom)
}
```

我们对每个子节点递归调用。

```javascript
function render(element, container) {
  const dom = document.createElement(element.type)
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
​
```

同时我们还需要操作文本元素，如果我们的元素的类型是 `TEXT_ELEMENT` 的话，我们就创建一个文本节点来代替常规节点。

```javascript
function render(element, container) {
  const dom = element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
```

最后，我们需要将元素的属性分配给节点

```javascript
function render(element, container) {
  const dom = element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
  
  const isProperty = key => key !== "children"
  
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
    
  element.props.children.forEach(child =>
    render(child, dom)
  )

  container.appendChild(dom)
}
```

到目前为止，我们就拥有了一个可以渲染 JSX 到 DOM 的库了

```jsx
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object"
          ? child
          : createTextElement(child)
      ),
    },
  }
}
​
function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: [],
    },
  }
}
​
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
​
const Didact = {
  createElement,
  render,
}
​
/** @jsx Didact.createElement */
const element = (
  <div id="foo">
    <a>bar</a>
    <b />
  </div>
)
const container = document.getElementById("root")
Didact.render(element, container)
```

## 第 3 步，并发模式

不过...在我们添加更多代码之前，我们需要重构一下。

递归调用 `render` 函数会有些问题。

一旦们开始渲染，我们在完成渲染所有元素树之前讲不会停下来。如果元素树特别大，将会阻塞主线程很长的时间。而且，如果浏览器需要执行诸如处理用户输入或使动画保持平滑等高优先级的工作，则它必须等待渲染完成。

```javascript
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
```

所以我们需要将这个工作拆到小的工作单元里去，这样我们可以在完成每个小单元工作的时候，暂停浏览器的渲染过程，以便处理其他需要做的事情。

```javascript
let nextUnitOfWork = null
​
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
function performUnitOfWork(nextUnitOfWork) {
  // TODO
}
```

我们使用了 `requestIdleCallback` 来进行循环。你可以将它认为成 `setTimeout`, 只是与指定时间不同，浏览器会在主线程待机的时候执行 callback。

> React 不再使用`requestIdleCallback` 。现在它使用了  *[scheduler package](https://github.com/facebook/react/tree/master/packages/scheduler)* ，但在这个例子中原理是相同的。

`requestIdleCallback` 可以传入一个 `deadline` 参数。我们可以用它来检查在浏览器需要再次操作之前，有多少空闲的时间。

> 在 2019 年 11 月以前，并发模式在 React 中并不是稳定版本。在稳定版本中的循环看起来更像是下面这样
>
> ```javascript
> while (nextUnitOfWork) {    
> nextUnitOfWork = performUnitOfWork(   
>  nextUnitOfWork  
> ) 
> }
> ```

在使用这个循环之前，我们需要设置第一个工作单元，写出 `performUnitOfWork` 函数，来执行当前的工作，并总是返回下一个工作单元。

## 第 4 步，Fiber

为了管理这些工作单元，我们需要一个数据结构：fiber 树

每一个 fiber 会对应一个元素，并且每个 fiber 都是一个工作单元。

通过例子来看一下：

假设我们想要渲染一个如下元素树：

```jsx
Didact.render(
  <div>
    <h1>
      <p />
      <a />
    </h1>
    <h2 />
  </div>,
  container
)
```

![Fiber Tree 0](https://pomb.us/static/de664d437c94d478778b965c66c91f99/ac667/fiber0.png)

在 `render` 中，我们将创建一个根 fiber 并将它设置为 `nextUnitOfWork`。剩下的工作将会在 `performUnitOfWork` 函数中发生，我们需要对每个 fiber 做三件事：

1. 将元素添加到 DOM
2. 为子元素创建 fibers
3. 选择下一个工作节点

这个数据结构的目的是为了帮助更容易地找到下一个工作单元。这也是为什么每个 fiber 都有一个链接到它的第一个子元素、下一个旁元素以及父元素。

![Fiber Tree 1](https://pomb.us/static/a88a3ec01855349c14302f6da28e2b0c/ac667/fiber1.png)

当我们在 fiber 上完成工作的时候，如果有一个 `child` fiber，这个 fiber 就会成为下个工作单元。

在我们的例子中，当我们在 `div` fiber 中工作时，下一个工作单元将会是 `h1` fiber。

![Fiber Tree 2](https://pomb.us/static/c1105e4f7fc7292d91c78caee258d20d/ac667/fiber2.png)

如果 fiber 没有 `child`，将会使用 `sibling` 作为下一个工作单元。

举个例子，`p` fiber 没有 `child`，所以在其完成后，会执行 `a` fiber。

![Fiber Tree 3](https://pomb.us/static/c8bdcc17706e9ab06233c980ed9cf007/ac667/fiber3.png)

而如果 fiber 没有 `child` 和 `siblig`，我们会到它的“叔叔”节点，`parent` 的 `sibling` 上面。就像例子中的 `a` 和  `h2` fiber 一样。

![Fiber Tree 4](https://pomb.us/static/19c304dcb3824b14722691ded539ecdb/ac667/fiber4.png)

同时，如果 `parent` 没有 `sibling`， 就会继续寻找上一级 `parent`，知道找到一个 `sibling` 或者到的根节点。如果到达了根节点，这意味着完成了这次 `render` 中所有工作。

接下来，让我们将它放到代码中。

首先我们移除掉 `render` 函数中的代码。

```javascript
function render(element, container) {
  const dom =
    element.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(element.type)
​
  const isProperty = key => key !== "children"
  Object.keys(element.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = element.props[name]
    })
​
  element.props.children.forEach(child =>
    render(child, dom)
  )
​
  container.appendChild(dom)
}
​
let nextUnitOfWork = null
```

保留原来函数中创建 DOM 节点那一部分，在之后会用到

```javascript
function createDom(fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type)
​
  const isProperty = key => key !== "children"
  Object.keys(fiber.props)
    .filter(isProperty)
    .forEach(name => {
      dom[name] = fiber.props[name]
    })
​
  return dom
}
​
function render(element, container) {
  // TODO set next unit of work
}
​
let nextUnitOfWork = null
```

在 `render` 函数中，我们设置 `nextUnitOfWork` 到 fiber 树的根节点。

```javascript
function render(element, container) {
  nextUnitOfWork = {
    dom: container,
    props: {
      children: [element],
    },
  }
}
​
let nextUnitOfWork = null
```

接着，当浏览器就绪，它会调用我们的 `workLoop` 并在根节点上开始工作。

```javascript
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
function performUnitOfWork(fiber) {
  // TODO add dom node
  // TODO create new fibers
  // TODO return next unit of work
}
```

首先，我们创建一个新的节点加到 DOM 上。

我们在 `fiber.dom` 这个属性上来追踪这个 DOM 节点。

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  // TODO create new fibers
  // TODO return next unit of work
}
```

然后每一个子元素创建一个新的 fiber。

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
 const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
  }
  // TODO return next unit of work
}
```

接着我们基于它是否是第一个子元素，往 fiber 树中设置子元素或者兄弟元素的属性。

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
 const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
  }
  if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
}
```

最后，就完成了 `performUnitOfWork` 函数

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
​
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
​
    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
​
```

## 第 5 步，渲染和提交阶段

我们还有另外一个问题。

我们在每次在某个元素上工作的时候，都添加了一个节点到 DOM 上。但是记住，浏览器可能在我们渲染完整颗树的时候就会中断。在这种情况下，用户会看到一个没有完成的 UI，我们并不希望这种情况出现。

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  if (fiber.parent) {
    fiber.parent.dom.appendChild(fiber.dom)
  }
    
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​...
}
```

所以，我们需要移除这块的 DOM 变化。

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
 const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​...
}
```

然后，我们将在 fiber 树的根节点进行追踪。我们把这个工作的根节点叫做 `wipRoot`

```javascript
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  }
  nextUnitOfWork = wipRoot
}
​
let nextUnitOfWork = null
let wipRoot = null
```

当我们完成所有工作（我们知道下一个单元不会工作）我们就将整个 fiber 树提交给了 DOM。

```javascript
function commitRoot() {
  // TODO add nodes to dom
}
​
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
  }
  nextUnitOfWork = wipRoot
}
​
let nextUnitOfWork = null
let wipRoot = null
​
function workLoop(deadline) {
  let shouldYield = false
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(
      nextUnitOfWork
    )
    shouldYield = deadline.timeRemaining() < 1
  }
​
  if (!nextUnitOfWork && wipRoot) {
    commitRoot()
  }
​
  requestIdleCallback(workLoop)
}
​
requestIdleCallback(workLoop)
​
```

我们在 `commitRoot` 函数中完成这一切，在这里我用递归的方法添加了所有节点到 DOM 之上。

```javascript
function commitRoot() {
  commitWork(wipRoot.child)
  wipRoot = null
}
​
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

## 第 6 步，重渲染

目前为止，我们只是把元素添加到了 DOM 节点上，但如果需要更新或者删除节点呢？

这就是我们将要做的事，我们需要修改 `render` 函数接收到的值，使其成为我们最后一次向 DOM 提交的 fiber 树。

所以在每次提交之后，我们需要保存“最后一次提交给 DOM 的 fiber 树”的引用。我们称它为 `currentRoot`。

同时，我们给每个 fiber 都添加了 `alternate` 属性。这个属性是我们之前上一次提交给 DOM 的旧 fiber 的一个链接。

```javascript
function commitRoot() {
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}
​
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
​
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
    alternate: currentRoot,
  }
  nextUnitOfWork = wipRoot
}
​
let nextUnitOfWork = null
let currentRoot = null
let wipRoot = null

```

先在，我们从 `performUnitOfWork` 的代码中抽取创建新 fiber 的部分...

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  const elements = fiber.props.children
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: fiber,
      dom: null,
    }
​
    if (index === 0) {
      fiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
​
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
```

到新的 `reconcileChildren` 函数中

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  const elements = fiber.props.children
  reconcileChildren(fiber, elements)
​
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
​
function reconcileChildren(wipFiber, elements) {
  let index = 0
  let prevSibling = null
​
  while (index < elements.length) {
    const element = elements[index]
​
    const newFiber = {
      type: element.type,
      props: element.props,
      parent: wipFiber,
      dom: null,
    }
​
    if (index === 0) {
      wipFiber.child = newFiber
    } else {
      prevSibling.sibling = newFiber
    }
​
    prevSibling = newFiber
    index++
  }
}
```

在这里，我们把旧的 fiber 和新的元素进行重渲染。

我们同时遍历旧的 fiber (wipFiber.alternate) 的子元素以及所有需要重渲染的元素集合。

如果我们忽略同时遍历数组和链表所有的样板，我们留下来了在循环中最重要的东西 `oldFiber` 和 `element`。**这个 `element` 就是我们想要渲染到 DOM 上的东西，而 `oldFiber` 是上一次已经渲染了的。**

```javas
function reconcileChildren(wipFiber, elements) {
  let index = 0
  let oldFiber =
    wipFiber.alternate && wipFiber.alternate.child
  let prevSibling = null
​
  while (
    index < elements.length ||
    oldFiber != null
  ) {
    const element = elements[index]
    let newFiber = null
​
    // TODO compare oldFiber to element
```

我们如下地比较它们：

- 如果老的 fiber 和新的 element 拥有相同的类型，我们就让 DOM 节点只更新参数。
- 如果和新节点的类型不同，以为着我们需要创建新的 DOM 节点。
- 如果类型不同，并且有一个老的 fiber，我们需要移除掉老的节点。

React 还是用了 `keys` ，可以更好的重渲染。举个例子，它可以探测到数组中的元素更换了位置。

```javascript
const sameType =
      oldFiber &&
      element &&
      element.type == oldFiber.type
​
    if (sameType) {
      // TODO update the node
    }
    if (element && !sameType) {
      // TODO add this node
    }
    if (oldFiber && !sameType) {
      // TODO delete the oldFiber's node
    }
```

当老的 fiber 和元素类型相同，我们创建一个新的 fiber ，使得 DOM 节点是从老的 fiber 中得到，而属性是从元素中得到的。

我们还给 fiber 加了新的属性：`efectTag`  我们会在之后的提交阶段用到它。

```javascript
  const sameType =
      oldFiber &&
      element &&
      element.type == oldFiber.type
​
    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: wipFiber,
        alternate: oldFiber,
        effectTag: "UPDATE",
      }
    }
```

如果遇到元素需要创建一个新 DOM 节点的情况，我们将新的 fiber 的 effect tag 标记成 `PLACEMENT`

```javascript
   if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: wipFiber,
        alternate: null,
        effectTag: "PLACEMENT",
      }
    }
```

当遇到需要删除节点的情况，我们不需要新的 fiber，于是把 effect tag 加到旧的 fiber 上。

```javascript
  if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION"
      deletions.push(oldFiber)
    }
```



但是当我们在我们把 fiber 树提交给 DOM 的时候，我们并没有旧的 fiber。

所以我们需要一个数组来追踪哪些节点需要移除。

```javascript
function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element],
    },
    alternate: currentRoot,
  }
  deletions = []
  nextUnitOfWork = wipRoot
}
​
let nextUnitOfWork = null
let currentRoot = null
let wipRoot = null
let deletions = null
```

这样，当我们提交 DOM 更改的时候，我们可以使用这个数组里的 fibers。

```javascript
function commitRoot() {
  deletions.forEach(commitWork)
  commitWork(wipRoot.child)
  currentRoot = wipRoot
  wipRoot = null
}
```

现在，可以更改 `commitWork` 函数，来处理不同的 `effectTags`

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  domParent.appendChild(fiber.dom)
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

如果 fiber 有 `PLACEMENT` 的 effect tag， 我们和之前一样添加 DOM 节点到父节点之上。

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

如果是 `DELETION`，就移除掉子节点

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  } else if (fiber.effectTag === "DELETION") {
    domParent.removeChild(fiber.dom)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

如果是 `UPDATE`, 我们就需要在已有的 DOM 节点上更新属性。

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
  const domParent = fiber.parent.dom
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  } else if (
    fiber.effectTag === "UPDATE" &&
    fiber.dom != null
  ) {
    updateDom(
      fiber.dom,
      fiber.alternate.props,
      fiber.props
    )
  } else if (fiber.effectTag === "DELETION") {
    domParent.removeChild(fiber.dom)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

接着来完成 `updateDom` 函数

```javascript
function updateDom(dom, prevProps, nextProps) {
  // TODO
}
```

我们比较老的 fiber 和新的 fiber 之间的属性，去除掉旧的属性，设置新的属性

```javascript
const isProperty = key => key !== "children"
const isNew = (prev, next) => key =>
  prev[key] !== next[key]
const isGone = (prev, next) => key => !(key in next)
function updateDom(dom, prevProps, nextProps) {
  // Remove old properties
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach(name => {
      dom[name] = ""
    })
​
  // Set new or changed properties
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name]
    })
}
```

有一种特殊的属性需要更新就是事件监听器，所以如果属性名以 "on" 开头的话，我们会另外处理：

```javascript
const isEvent = key => key.startsWith("on")
const isProperty = key =>
  key !== "children" && !isEvent(key)
```

如果事件处理函数变换了的话，我们将它从节点上移除

```javascript
//Remove old or changed event listeners
  Object.keys(prevProps)
    .filter(isEvent)
    .filter(
      key =>
        !(key in nextProps) ||
        isNew(prevProps, nextProps)(key)
    )
    .forEach(name => {
      const eventType = name
        .toLowerCase()
        .substring(2)
      dom.removeEventListener(
        eventType,
        prevProps[name]
      )
    })
```

然后添加新的处理函数

```javascript
 // Add event listeners
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      const eventType = name
        .toLowerCase()
        .substring(2)
      dom.addEventListener(
        eventType,
        nextProps[name]
      )
    })
```

可以在 [codesandbox](https://codesandbox.io/s/didact-6-96533) 中尝试到目前为止的代码。

## 第 7 步，函数组件

接下来我们要做的是添加对函数组件的支持

首先让我们把例子改一下。我们使用一个新的函数组件，返回一个 `h1` 元素。

```jsx
/** @jsx Didact.createElement */
function App(props) {
  return <h1>Hi {props.name}</h1>
}
const element = <App name="foo" />
const container = document.getElementById("root")
Didact.render(element, container)

```

注意如果我们如果把 jsx 翻译成 js，它会是这样：

```javascript
function App(props) {
  return Didact.createElement(
    "h1",
    null,
    "Hi ",
    props.name
  )
}
const element = Didact.createElement(App, {
  name: "foo",
})
```

函数组件有两点不同：

- 从函数组件来的 fiber 没有 DOM 节点
- 子元素是从运行中的函数得来的，而不是直接在 `props` 中传入

```javascript
function performUnitOfWork(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
​
  const elements = fiber.props.children
  reconcileChildren(fiber, elements)
  ...
}
```

当我检查到 fiber 的类型是函数的话，我们需要一个不同的更新函数。

在 `updateHostComponent` 中，我们做和以前相同的事：

```javascript
function performUnitOfWork(fiber) {
  const isFunctionComponent =
    fiber.type instanceof Function
  if (isFunctionComponent) {
    updateFunctionComponent(fiber)
  } else {
    updateHostComponent(fiber)
  }
  if (fiber.child) {
    return fiber.child
  }
  let nextFiber = fiber
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling
    }
    nextFiber = nextFiber.parent
  }
}
​
function updateFunctionComponent(fiber) {
  // TODO
}
​
function updateHostComponent(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber)
  }
  reconcileChildren(fiber, fiber.props.children)
}
​
```

而在 `updateFunctionComponent` 中，我们运行函数来获得子元素。

举例来说，这里的 `fiber.type` 是 `App` 函数，当我们运行它时，返回 `h1` 元素。

然后，当我们获得了子元素吼，重渲染过程会回到之前同样的方式上，我们不需要修改其他的东西。

```javascript
function updateFunctionComponent(fiber) {
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}
```

我们还需要修改一下 `commitWork` 函数。

在遇到没有 DOM 的 fibers 的情况下，我们需要改两件事：

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
​
  const domParent = fiber.parent.dom
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  } else if (
    fiber.effectTag === "UPDATE" &&
    fiber.dom != null
  ) {
    updateDom(
      fiber.dom,
      fiber.alternate.props,
      fiber.props
    )
  } else if (fiber.effectTag === "DELETION") {
    domParent.removeChild(fiber.dom)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
​
```

首先，去找到父元素的 DOM 节点，我们需要向上查找 fiber 树，直到找到一个有 DOM 节点的 fiber.

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
​
 let domParentFiber = fiber.parent
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent
  }
  const domParent = domParentFiber.dom
  
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  } else if (
    fiber.effectTag === "UPDATE" &&
    fiber.dom != null
  ) {
    updateDom(
      fiber.dom,
      fiber.alternate.props,
      fiber.props
    )
  } else if (fiber.effectTag === "DELETION") {
    domParent.removeChild(fiber.dom)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}
```

然后当移除节点的时候，我们同样需要同样的搜寻直到找到有 DOM 节点的子元素。

```javascript
function commitWork(fiber) {
  if (!fiber) {
    return
  }
​
 let domParentFiber = fiber.parent
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent
  }
  const domParent = domParentFiber.dom
  
  if (
    fiber.effectTag === "PLACEMENT" &&
    fiber.dom != null
  ) {
    domParent.appendChild(fiber.dom)
  } else if (
    fiber.effectTag === "UPDATE" &&
    fiber.dom != null
  ) {
    updateDom(
      fiber.dom,
      fiber.alternate.props,
      fiber.props
    )
  } else if (fiber.effectTag === "DELETION") {
    commitDeletion(fiber, domParent)
  }
​
  commitWork(fiber.child)
  commitWork(fiber.sibling)
}

function commitDeletion(fiber, domParent) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom)
  } else {
    commitDeletion(fiber.child, domParent)
  }
}
```

## 第 8 步， Hooks

最后，在我们拥有了函数组件之后，让我们添加一些状态。

让我们改变以下例子，以一个经典的 Counter 组件为例。当我们每次点击的时候，状态都会加一。

注意我们现在使用 `Didact.useState` 来获取和更新计数器的值。

```javascript
const Didact = {
  createElement,
  render,
  useState,
}
​
/** @jsx Didact.createElement */
function Counter() {
  const [state, setState] = Didact.useState(1)
  return (
    <h1 onClick={() => setState(c => c + 1)}>
      Count: {state}
    </h1>
  )
}
const element = <Counter />
const container = document.getElementById("root")
Didact.render(element, container)
```

这里是我们在 `Counter` 函数中调用的函数 `useState`

```javascript
function updateFunctionComponent(fiber) {
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}
​
function useState(initial) {
  // TODO
}
```

在调用我们的函数组件之前，我们需要初始化一些全局变量，这样我们就可以在 `useState` 函数中使用它们

首先我们把 fiber 初始化。

同时我们添加了一个 `hooks` 的数组到 fiber 上来支持我们在同一个组件里多次调用 `useState`。然后我们追踪当前 hook 的索引。

```javascript
let wipFiber = null
let hookIndex = null
​
function updateFunctionComponent(fiber) {
  wipFiber = fiber
  hookIndex = 0
  wipFiber.hooks = []
  const children = [fiber.type(fiber.props)]
  reconcileChildren(fiber, children)
}
​
function useState(initial) {
  // TODO
}
```

当函数组件调用 `useState`，我们检查是否有老的 hook， 可以通过 fiber 的 `alternate` 属性以及 hook 索引来查找。

如果有一个老的 hook，我们就把老的 hook 里的状态拷贝到新的 hook 里去，如果没有的话，就填入初始值。

然后我们把新的 hook 添加到 fiber 上，把  hook 的索引加一，返回状态。

```javascript
function useState(initial) {
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex]
  const hook = {
    state: oldHook ? oldHook.state : initial,
  }
​
  wipFiber.hooks.push(hook)
  hookIndex++
  return [hook.state]
}
```

`useState` 还应该返回一个函数来更新状态，所以我们定义一个 `setState` 函数来接收一个动作（在 `Counter` 的例子中，这个动作是让 state 加一的函数）。

我们将这个动作 push 到一个队列中，并添加到 hook 上。

然后我们做和 `render` 函数中类似的事，设置一个新的根到下一个工作单元，所以就可以重新渲染了。

```javascript
function useState(initial) {
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex]  
  
  const hook = {
    state: oldHook ? oldHook.state : initial,
    queue: [],
  }
​
  const setState = action => {
    hook.queue.push(action)
    wipRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    }
    nextUnitOfWork = wipRoot
    deletions = []
  }
​
  wipFiber.hooks.push(hook)
  hookIndex++
  return [hook.state, setState]
}
```

但我们还没有运行这个动作。

我们在组件进行下一次渲染时做这个动作，从老的 hook 队列中得到所有动作，然后挨个执行他们到新的 hook 状态上，这样当我们返回的时候，状态是更新了的。

```javascript
function useState(initial) {
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex]  
  
  const hook = {
    state: oldHook ? oldHook.state : initial,
    queue: [],
  }
  
  const actions = oldHook ? oldHook.queue : []
  actions.forEach(action => {
    hook.state = action(hook.state)
  })
​
  const setState = action => {
    hook.queue.push(action)
    wipRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot,
    }
    nextUnitOfWork = wipRoot
    deletions = []
  }
​
  wipFiber.hooks.push(hook)
  hookIndex++
  return [hook.state, setState]
}
```

这就是所有。我们创建了一个自己版本的 React。

你可以在  [codesandbox](https://codesandbox.io/s/didact-8-21ost) 或者 [github](https://github.com/pomber/didact).上来尝试它。

```javascript
function createElement(type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.map(child =>
        typeof child === "object" ? child : createTextElement(child)
      )
    }
  };
}

function createTextElement(text) {
  return {
    type: "TEXT_ELEMENT",
    props: {
      nodeValue: text,
      children: []
    }
  };
}

function createDom(fiber) {
  const dom =
    fiber.type == "TEXT_ELEMENT"
      ? document.createTextNode("")
      : document.createElement(fiber.type);

  updateDom(dom, {}, fiber.props);

  return dom;
}

const isEvent = key => key.startsWith("on");
const isProperty = key => key !== "children" && !isEvent(key);
const isNew = (prev, next) => key => prev[key] !== next[key];
const isGone = (prev, next) => key => !(key in next);
function updateDom(dom, prevProps, nextProps) {
  //Remove old or changed event listeners
  Object.keys(prevProps)
    .filter(isEvent)
    .filter(key => !(key in nextProps) || isNew(prevProps, nextProps)(key))
    .forEach(name => {
      const eventType = name.toLowerCase().substring(2);
      dom.removeEventListener(eventType, prevProps[name]);
    });

  // Remove old properties
  Object.keys(prevProps)
    .filter(isProperty)
    .filter(isGone(prevProps, nextProps))
    .forEach(name => {
      dom[name] = "";
    });

  // Set new or changed properties
  Object.keys(nextProps)
    .filter(isProperty)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      dom[name] = nextProps[name];
    });

  // Add event listeners
  Object.keys(nextProps)
    .filter(isEvent)
    .filter(isNew(prevProps, nextProps))
    .forEach(name => {
      const eventType = name.toLowerCase().substring(2);
      dom.addEventListener(eventType, nextProps[name]);
    });
}

function commitRoot() {
  deletions.forEach(commitWork);
  commitWork(wipRoot.child);
  currentRoot = wipRoot;
  wipRoot = null;
}

function commitWork(fiber) {
  if (!fiber) {
    return;
  }

  let domParentFiber = fiber.parent;
  while (!domParentFiber.dom) {
    domParentFiber = domParentFiber.parent;
  }
  const domParent = domParentFiber.dom;

  if (fiber.effectTag === "PLACEMENT" && fiber.dom != null) {
    domParent.appendChild(fiber.dom);
  } else if (fiber.effectTag === "UPDATE" && fiber.dom != null) {
    updateDom(fiber.dom, fiber.alternate.props, fiber.props);
  } else if (fiber.effectTag === "DELETION") {
    commitDeletion(fiber, domParent);
  }

  commitWork(fiber.child);
  commitWork(fiber.sibling);
}

function commitDeletion(fiber, domParent) {
  if (fiber.dom) {
    domParent.removeChild(fiber.dom);
  } else {
    commitDeletion(fiber.child, domParent);
  }
}

function render(element, container) {
  wipRoot = {
    dom: container,
    props: {
      children: [element]
    },
    alternate: currentRoot
  };
  deletions = [];
  nextUnitOfWork = wipRoot;
}

let nextUnitOfWork = null;
let currentRoot = null;
let wipRoot = null;
let deletions = null;

function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }

  if (!nextUnitOfWork && wipRoot) {
    commitRoot();
  }

  requestIdleCallback(workLoop);
}

requestIdleCallback(workLoop);

function performUnitOfWork(fiber) {
  const isFunctionComponent = fiber.type instanceof Function;
  if (isFunctionComponent) {
    updateFunctionComponent(fiber);
  } else {
    updateHostComponent(fiber);
  }
  if (fiber.child) {
    return fiber.child;
  }
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling;
    }
    nextFiber = nextFiber.parent;
  }
}

let wipFiber = null;
let hookIndex = null;

function updateFunctionComponent(fiber) {
  wipFiber = fiber;
  hookIndex = 0;
  wipFiber.hooks = [];
  const children = [fiber.type(fiber.props)];
  reconcileChildren(fiber, children);
}

function useState(initial) {
  const oldHook =
    wipFiber.alternate &&
    wipFiber.alternate.hooks &&
    wipFiber.alternate.hooks[hookIndex];
  const hook = {
    state: oldHook ? oldHook.state : initial,
    queue: []
  };

  const actions = oldHook ? oldHook.queue : [];
  actions.forEach(action => {
    hook.state = action(hook.state);
  });

  const setState = action => {
    hook.queue.push(action);
    wipRoot = {
      dom: currentRoot.dom,
      props: currentRoot.props,
      alternate: currentRoot
    };
    nextUnitOfWork = wipRoot;
    deletions = [];
  };

  wipFiber.hooks.push(hook);
  hookIndex++;
  return [hook.state, setState];
}

function updateHostComponent(fiber) {
  if (!fiber.dom) {
    fiber.dom = createDom(fiber);
  }
  reconcileChildren(fiber, fiber.props.children);
}

function reconcileChildren(wipFiber, elements) {
  let index = 0;
  let oldFiber = wipFiber.alternate && wipFiber.alternate.child;
  let prevSibling = null;

  while (index < elements.length || oldFiber != null) {
    const element = elements[index];
    let newFiber = null;

    const sameType = oldFiber && element && element.type == oldFiber.type;

    if (sameType) {
      newFiber = {
        type: oldFiber.type,
        props: element.props,
        dom: oldFiber.dom,
        parent: wipFiber,
        alternate: oldFiber,
        effectTag: "UPDATE"
      };
    }
    if (element && !sameType) {
      newFiber = {
        type: element.type,
        props: element.props,
        dom: null,
        parent: wipFiber,
        alternate: null,
        effectTag: "PLACEMENT"
      };
    }
    if (oldFiber && !sameType) {
      oldFiber.effectTag = "DELETION";
      deletions.push(oldFiber);
    }

    if (oldFiber) {
      oldFiber = oldFiber.sibling;
    }

    if (index === 0) {
      wipFiber.child = newFiber;
    } else if (element) {
      prevSibling.sibling = newFiber;
    }

    prevSibling = newFiber;
    index++;
  }
}

const Didact = {
  createElement,
  render,
  useState
};

/** @jsx Didact.createElement */
function Counter() {
  const [state, setState] = Didact.useState(1);
  return (
    <h1 onClick={() => setState(c => c + 1)} style="user-select: none">
      Count: {state}
    </h1>
  );
}
const element = <Counter />;
const container = document.getElementById("root");
Didact.render(element, container);

```

## 结语

在帮助你理解 React 如何工作的同时，这篇文章的另一个目的是让你可以更容易的理解 React 代码更深的东西。这也是为什么我们几乎使用了同样的变量名与函数名。

举个i，如果你在真正的 React 应用中的函数组件中添加了断点，调用栈会是这样：

- `workLoop`
- `performUnitOfWork`
- `updateFunctionComponent`

我们没有添加更多的 React 功能和优化。比如说，这些东西 React 做的就不一样：

- 在 Diadact 中，我们在渲染阶段遍历了整颗树。而 React 会有一些方法来跳过完全没有变化的子树。
- 我们在提交阶段同样遍历了整棵树。React 则维护了一个链接列表，来管理拥有副作用的 fibers，并且只监视这些 fibers。
- 每次在树中创建新的工作的时候，我们都为每一个 fiber 创建了一个新的对象。React 则会从之前的树上回收 fibers。
- 当 Didact 在渲染阶段接收到新的更新，将会抛出一个工作给树并且从根开始渲染。React 则用超时时间戳来标记每个更新，并用它来决定哪个更新有更高的优先级。
- 更多...

你还可以添加一些功能：

- 用一个对象来添加 style 属性
- 拉平 children 数组
- useEffect hook
- 利用 key 来重渲染。

如果你给 Diadact 添加了如下功能，可以到  [GitHub repo](https://github.com/pomber/didact) 提交一个 PR，这样别人也就能看到了~

感谢你的阅读！
