---
title: React“顿悟”瞬间
date: 2017-01-23 23:11:08
tags:
  - 翻译
  - React
---

# React“顿悟”瞬间

[原文地址](https://medium.freecodecamp.com/react-aha-moments-4b92bd36cc4e#.ovauy7m5h)

作为一名老师，我的主要目标之一是最大化学生们的“顿悟”瞬间。

一个“顿悟”瞬间是指突然领悟或者明白的瞬间，在那一瞬间主题开始显得更加合理。我们每个人都经历过这样的瞬间。并且我所知道的最优秀的老师都懂得去理解他们的听众以及调整课程以最大化这样的瞬间。

在过去的几年里，我通过各种流行的媒介教授React。自始至终，我都详细记录了什么触发了这样的瞬间，尤其是学习React的时候。

大约两周前，我看到了[Reddit上的一篇贴子](https://www.reddit.com/r/reactjs/comments/5gmywc/what_were_the_biggest_aha_moments_you_had_while/)，这篇帖子描述了相似的观点。受它激发，我写下了这篇包含了私人收藏的这样的瞬间以及别人在那篇Reddit贴子里分享的其他的瞬间。我希望这能有助于你用新的方式来理解React的概念。

## 观点1：组件的Props就是函数的参数

React的优点之一是你可以使用你已经拥有的对待javascript函数的直觉，然后使用它来决定在何时何地你应该创建一个React组件。不过在React，你的函数接受一些参数然后返回UI的一个对象表示，而不是接受一些参数然后返回一个值。

这个想法可以用如下的公式总结：`fn(d)=V`，该公式读做：“一个函数接受一些数据然后返回一个视图”。

这是一种看待UI界面开发的优雅的方式，因为这时你的UI不过是一系列不同函数调用的组合。这就回到了你已经熟悉的构建应用程序的方式，并且此时你可以在编写UI时利用函数组合带来的所有好处。

## 观点2：在React，你的应用的所有UI都是构建于函数组合上的，JSX不过是在这些函数之上的一层抽象

我见到的初学者对React最常见的反应是：“React看起来很酷，但是我不喜欢JSX。因为它破坏了关注点分离原则”。

JSX并不是尝试去替代HTML，并且它也绝对不仅仅是一门模板语言。

理解JSX需要注意两个重要的点。

第一，[JSX是函数`React.createElement`的抽象](https://tylermcginnis.com/react-elements-vs-react-components/)，那个函数用于返回DOM的一个对象表示。

我知道这听起来很啰嗦，基本上无论何时你写了JSX，一旦它被编译，你将得到一个表示实际DOM的javascript对象，或者无论何种表示你当前平台（IOS，Android，等等）的视图。接着做一次对比，React可以仅在有变化时更新DOM。

这带来一些性能上的优势，但更重要的是它显示了JSX真的“仅仅是Javascript”。

第二，由于JSX就是Javascript，因此你能获得Javascript提供的所有好处，比如组合，lint以及debugging。并且你仍然能使用你已经熟悉的HTML，以及它的声明式的语法。

## 观点3：组件并不一定需要与DOM节点匹配

在你最开始学习React的时候，你被教导“组件是React的构建模块。它们接受输入并返回UI（Descriptor模式）”。

这是否意味着每个组件都应该像我们通常被教导的那样返回UI描述符呢？如果我们想要一个组件渲染另外一个组件呢（高阶组件模式）？如果我们想要一个组件管理一些状态，但是不返回UI描述符，而是返回以state为参数的函数调用（Render Props模式）？如果我们希望一个组件负责管理声音而不是视觉UI，那么它该返回什么？

React很棒的一点是你的组件并不*必须*返回通常意义的“视图”。只要最终返回结果是一个React element，null或者false之一就行。

你可以返回其他组件：

```js
render () { 
  return <MyOtherComponent /> 
}
```

你可以返回函数调用：

``` js
render () { 
  return this.props.children(this.someImportantState) 
}
```

或者你可以什么都不返回：

```js
render () { 
  return null 
}
```

Ryan Florence的[React Rally talk](https://www.youtube.com/watch?v=kp-NOggyz54)对这个原则进行了更深入的介绍，我非常喜欢这个视频。

## 观点4：当两个组件需要共享state，我们需要提升那个state而不是尝试去保持state间的同步

自然地，基于组件的结构会使状态共享变得更加困难。如果两个组件共享同一个state，那么这个state该放在哪里呢？

这是一个如此常见的问题，以致于它催生了一个解决方案的完整生态系统，最终以Redux结束。

Redux的方案是把那个共享的state放到一个叫做“store”的地方。组件可以订阅store的任何所需的部分，然后通过分发“actions”来更新store。

React的方案是找到那些组件的最近共同父组件，让那个父组件维护共享的state，然后向下传给需要的子组件。这两种方法都有各自的优势和劣势，重要的是我们要清楚有这些方案存在。

## 观点5：在React里继承不是必需的，并且隔离和特化都可以通过组合实现

React一直都是（在好的意义上）非常慷慨地采用函数式编程原则。React远离继承并且向组合靠拢的例子之一是在发布0.13版本的时候没有为ES类添加mixin支持。

这么做的原因是大多数你能通过mixin（或者继承）做到的事，你也可以通过组合来完成，并且副作用更少。

如果你是伴随着很深继承思想来到React，那么这些新的思考方式也行感觉会更加困难或者不自然。幸运的是有许多资源可以用来帮助我们。比如我非常喜欢[这个视频](https://www.youtube.com/watch?v=wfMtDGfHWpA)，并且它不特定于React。

## 观点6：分离容器组件和展示组件

如果用解刨学的观点看React组件，它通常包含了一些state，可能还有些生命周期事件，以及JSX的代码。

假如不在一个组件里面包含全部的这些东西，你可以将state以及生命周期事件和JSX代码分离。这会得到两个组件。第一个包含state以及生命周期事件，它负责组件应如何工作。第二个通过props接受数据，它负责组件应如何展示。

这种方式能给展示组件带来更高的复用性，因为它们不再和需要的数据耦合在一起。

我还发现这种方式能使你（和新进你项目的同事）更好地理解项目结构。你可以在不看到或者关系UI的情况下交换组件的实现，反之亦然。结果是设计师可以调整UI而不需要担心这些展示组件如何接收数据。

查看[展示和容器组件](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.q9tui51xz)获取更详细信息。

## 观点7：如果你尽可能保持你的组件纯净和无状态，那么维护将变得更容易

这是讲展示组件和容器组件分离的另外一个好处。

state伴随着不一致。通过合理的分离，将复杂度封装，你能极大地提升你的应用的可预测性。

多谢花时间阅读我的这篇文章。你可以在[Twitter上follow我](https://twitter.com/tylermcginnis33)或者[查看我的博客](https://tylermcginnis.com/react-aha-moments/)获取更多关于JavaScript和React的观点。
