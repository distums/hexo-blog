---
title: 如何使用React的高阶组件
date: 2017-02-15 22:20:57
tags:
  - 翻译
  - React
---

# 如何使用React的高阶组件

[原文地址](https://medium.freecodecamp.com/react-higher-order-components-635d0bc38b6c#.i5ds50mvo)

伴随着React开始进入公众视野，它带来了新的开发前端架构的方式。它被认为是MVC模式的“View”层。

一直以来，container/presentational模式都倍受React生态的核心贡献者推崇。

在这篇文章，你将学习更多关于container/presentational模式的知识，以及如何使用它来创建React组件。

在开始前，让我们先阐明一些术语：

- 一个container是一个获取数据并且（或者）转换数据的组件。

- 一个presentational 组件展示来自props的数据。没有其他的了。

- 一个container充当一个高阶组件。这里结合使用高阶组件可以将这两个层隔离开来。这可以防止你在presentational层混入不需要的逻辑。

如果你有意愿，你可以一边用[ESNextbin](https://esnextb.in/)进行演练。

跳转到[ESNextbin](https://esnextb.in/)页面后，在它顶部你可以看到3个Tab：**CODE**，**HTML**和**PACKAGE**。点击**PACKAGE**，切换到该Tab后添加依赖*react*和*react-dom*。

现在点击**CODE**，然后引入*react*以及*react-dom*的**render**。

```js
import { default as React } from ‘react’
import { render } from ‘react-dom’
```

现在让我们创建一个只渲染“hello,world!”的stateless functional组件。

```js
const Presentational = () => <div>Hello, world!</div>
```

之后，你仅需使用*render*将该组件挂载到DOM上。

```js
render(<Presentational />, document.querySelector(‘#root’))
```

当然现在你还没在**HTML**里面创建**root**节点，让我们现在进行。所有你需要做的就是添加一个*id*是**root**的*div*。

```html
<!doctype html>
<html>
<head>
 <meta charset=”utf-8">
 <title>ESNextbin Sketch</title>
 <! — put additional styles and scripts here →
</head>
<body>
 // Notice something different
 <div id=‘root’></div>
</body>
</html>
```

现在是见证真理的时刻。点击页面右上角的**EXECUTE**。你应当看到屏幕上显示了*Hello,world!*。

现在你已经有了一个presentational组件，但你还需要一个container组件用于获取数据。

让我们在presentational组件前面创建一个container组件。为了创建改container组件，你需要使用React的Component类。因此，你需要在页面顶部引入它。

```js
import { default as React, Component } from ‘react’
```

现在开始你可以着手创建container组件了。

```js
class Container extends Component {
 
 state = {
   data: []
 }
 
 componentDidMount() {
   fetch(‘https://fcctop100.herokuapp.com/api/fccusers/top/alltime')
   .then(response => response.json())
   .then(data => this.setState({ data }))
 }
 
 render() {
   return <Presentational data={this.state.data} />
 }
}
```

在顶部，你有一个state，它是个空数组。你还使用了`componentDidMount`这个生命周期函数用于获取数据。

在得到响应后你用它返回的数据更新state。接着这个数据作为prop传到了presentational组件。这个意味着你需要升级presentational组件的代码。

```
const Presentational = ({ data }) => 
  <div>{JSON.stringify(data)}</div>
```

这里我使用解构从props对象取得**data**字段。否则它看起来将是这样的：

```js
const Presentational = props => 
  <div>{JSON.stringify(props.data)}</div>
```

这些改动迫使你要修改挂载的组件。现在你需要渲染container组件。

```js
render(<Container />, document.querySelector(‘#root’))
```

当你点击**EXECUTE**后你会看到一大堆的数据填满了你的屏幕。天啊！

这里你所做的事是抽出了container组件以及应当由它完成的事。但这不够灵活。你需要将presentational组件写到你的container组件里面。是否存在一种更加灵活的方式？正式进入高阶组件。

一个高阶组件是一个函数接收一个组件并返回一个新的组件。它实际看起来是怎样的。让我们新建一个函数接收一个组件并返回一个包含来自FCC API数据的新组件。你将更新container组件来完成这个功能。

```js
const container = Presentational => 
 class extends Component {
 
 state = {
   data: []
 }
 
 componentDidMount() {
   fetch(‘https://fcctop100.herokuapp.com/api/fccusers/top/alltime')
   .then(response => response.json())
   .then(data => this.setState({ data }))
 }
 
 render() {
   return <Presentational data={this.state.data} />
 }
}
```

这里并没有太大的不同。但还有一个额外的步骤需要完成。你需要将这个container组件和presentational组件组合在一起。我仅仅是使用presentational组件作为参数调用`container`函数，这仅需在presentational组件下面添加如下代码：

```js
const HigherOrderComponent = container(Presentational)
```

现在你需要做的只是渲染该高阶组件。

```js
render(<HigherOrderComponent />, document.querySelector(‘#root’))
```

你弄明白了这里你完成了什么吗？现在你对任何组件使用`container`，而不仅仅是那个presentational组件。

让我们更进一步。我们调用另一个函数告诉container组件从哪获取数据。我认为这会使这个container组件更具有灵活性。

新的container组件看起来是这样的：

```js
const container = endpoint => Presentational => 
 class extends Component {
 
 state = {
   data: []
 }
 
 componentDidMount() {
   fetch(endpoint)
     .then(response => response.json())
     .then(data => this.setState({ data }))
 }
 
 render() {
   return <Presentational data={this.state.data} />
 }
}
```

现在你需要更新高阶组件的代码来处理新的远端调用。

```js
const HigherOrderComponent = container(
  ‘https://fcctop100.herokuapp.com/api/fccusers/top/alltime'
)(Presentational)
```

If that looks a little off to you then you have good eye（这句看不懂囧）。假若你能进一步做抽象呢？转入*recompose*。

让我们先暂停一会讨论下compose的思想是什么。为了理解compose你先要理解mapping。

在计算机编程领域mapping指接收一个值并将它转成另一个值。基本上每个被创建出来的函数都是用来做这个的。看下下面的例子体会下：

```js
function double(x) { 
  return x * 2
}
const doubled = double(2)
```

假设现在我想创建另一个转换函数，比如3倍，我该如何做呢？

```js
function triple(x) { 
  return x * 3 
}

function double(x) { 
  return x * 2
}

const messedAroundAndGotATripleDouble = triple(double(2))
```

很简单，对吧？假如我想一个接一个地创建转换函数呢，函数调用会变得相当长。借助于compose函数你不但可以使代码更具可读性，而且组合性更高。

```js
function compose(f, g) { 
  return function(x) { 
    return f(g(x))
  }
}

function triple(x) { 
  return x * 3 
}

function double(x) { 
  return x * 2
}

const tripleThenDouble = compose(triple, double)

const messedAroundAndGotATripleDouble = tripleThenDouble(2)
```

这只是一个简单的例子，但是它展示了你可以如何对同一个初始值应用多个转换函数。

现在设想presentational组件就是初始值，而container组件是转换函数之一。大脑在颤抖了！

使用*recompose*前要先将*recompose*库拉下来。我们切到**PACKAGES**页下将*recompose*库添加进来。

```json
{
 “name”: “esnextbin-sketch”,
 “version”: “0.0.0”,
 “dependencies”: {
   “react”: “15.3.2”,
   “react-dom”: “15.3.2”,
   “recompose”: “latest”,
   “babel-runtime”: “6.11.6”
 }
}
```

返回到**CODE**页，引入**compose**。

```js
import { compose } from ‘recompose’
```

接着你就可以使用*compose*来进一步抽象高阶组件。

```js
const HigherOrderComponent = compose(
 container(
  ‘https://fcctop100.herokuapp.com/api/fccusers/top/alltime'
 )
)
```

并且你可以更加声明式的描述整个组件所做的事。

```js
const FetchAndDisplayFCCData = HigherOrderComponent(Presentational)

render(<FetchAndDisplayFCCData />, document.querySelector(‘#root’)
```

最终这允许你创建更加灵活多用的组件。你甚至可以与高阶组件分离单独测试presentational组件。高阶组件万岁！

为了好玩，让我们将presentational组件从渲染一大块数据改为渲染一组列表元素。

```js
const Presentational = ({ data }) => 
 <ul>
   {data.map((v, k) => 
     <li key={k}>
       {v.username}
     </li>
   )}
 </ul>
 ```
 
 哇哦！真是好多的内容。如果想看完整的代码你可以访问我创建的这个[gist](https://gist.github.com/arecvlohe/c5005643ea4fcb9637ccd9f60b98d305)。
 
 更多的内容可以访问Andrew Clark的这个叫*[recompose](https://github.com/acdlite/recompose)*的React工具库。
