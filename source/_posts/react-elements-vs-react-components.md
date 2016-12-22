---
title: React Elements VS React Components
date: 2016-12-21 22:28:18
tags:
  - react
  - 翻译
---

# React Elements VS React Components

> [原文地址](https://medium.freecodecamp.com/react-elements-vs-react-components-fdc776705880#.seeeu3k9g)

几个月前我在[Twitter](https://twitter.com/tylermcginnis33/status/771087982858113024)发了个我认为是简单的问题：

![](./react_elements_twitter.png)

使我吃惊的不是围绕这个问题的共同的困惑，而是我收到的不准确回答的数量。

> 实例/实例化
> 渲染
> 执行
> 调用
> “使用它:)”

产生困惑的首要原因是在JSX和React底层真正发生的存在一层常常未被触及的抽象层。为了回答这个问题，我们需要深入到这层抽象中。

让我们从审查React的根本开始。

## React究竟是什么

React是一个用于构建用户界面的库。无论React自身或者React生态看起来有多么复杂，这就是React的核心-构建UI。知道了这点，我们就能得到我们的第一个定义，*Element*。

简单来说，*一个React element描述你想在屏幕上看到什么*。

复杂点说，*一个React element是一个DOM节点的对象表示*。

注意这里我用了‘描述’这个词。非常重要的是一个React element并不就是你在屏幕上实际看到的东西，反而，只是它的对象表示。这是有原因的：

1. javascript对象是轻量的，React可以创建和销毁这些elements而不产生过多开销。

2. React可以分析这些对象，然后分析实际DOM元素，再然后当有改动产生时更新实际DOM元素。这能够带来一些性能方面的好处。

为了创建一个DOM节点的对象表示（即React element），我们可以使用React的`createElement`方法：

``` js
const element = React.createElement( 
  'div', 
  {id: 'login-btn'}, 
  'Login'
)
```

`createElement`接受3个参数：

1. tag名字字符串（div,span,等等）

2. 任何你想让该元素拥有的属性

3. 内容或者元素的子元素——对于这个例子是文本“Login”

上面的`createElement`调用会返回一个如下形式的对象：

``` js
{ 
  type: 'div', 
  props: { 
    children: 'Login', 
    id: 'login-btn' 
  } 
}
```

当它渲染到DOM中（使用ReactDOM.render）时，我们会得到一个新的看起来是这样的DOM节点：

``` html
<div id='login-btn'>Login</div>
```

到目前为止，一切都好。学习React有趣的地方在于你被教授的第一个东西往往是components。“Components是React的构建区块”。

然而，请注意我们以elements作为本篇博文的开始。因为，一旦你理解了elements，你将能平滑过渡到理解components。

*一个Component是一个接受可选输入并返回一个React element的函数或者类。*

``` js
function Button ({ onLogin }) { 
  return React.createElement( 
    'div', 
    {id: 'login-btn', onClick: onLogin}, 
    'Login' 
  )
}
```

按定义，我们有一个Button component接收onLogin输入并返回一个React element。值得注意的一件事是我们的Button component接收一个onLogin方法作为它的prop。为了把该方法一同传给我们的DOM元素的对象表示，我们将它传给了`createElement`方法的第二个参数，正如我们对id属性所做的一样。

让我们再深入些。

到这一刻，我们仅仅讨论了创建“type”属性为原生HTML元素（“span”,“div”,等等）的React element，但是你也可以传入其他React components作为`createElement`方法的第一个参数。

``` js
const element = React.createElement(
  User, 
  {name: 'Tyler McGinnis'},
  null 
)
```

然而，与一个HTML标签名不同，如果React看到第一个参数是一个类或者函数，接下来它将会给定相应的props去检查看看该类或者函数会渲染出什么elements。React会不断重复这个过程直到没有用一个类或者函数作为第一个参数的`creatElement`调用。让我们实际操作下看看。

``` js
function Button ({ addFriend }) {
  return React.createElement(
    "button", 
    { onClick: addFriend }, 
    "Add Friend" 
  ) 
} 

function User({ name, addFriend }) { 
  return React.createElement(
    "div", 
    null,
    React.createElement( "p", null, name ),
    React.createElement(Button, { addFriend })
  ) 
}
```

如上我们拥有两个component，一个Button和一个User。User对应DOM的对象表示将是一个包含两个子元素的“div”，一个包裹用户名的“p”节点和一个Button component。现在我们用返回值去替换createElement调用。

``` js
function Button ({ addFriend }) { 
  return { 
    type: 'button', 
    props: { 
      onClick: addFriend, 
      children: 'Add Friend' 
    } 
  } 
} 

function User ({ name, addFriend }) { 
  return { 
    type: 'div', 
    props: { 
      children: [{ 
        type: 'p',
        props: { children: name } 
      }, 
      { 
       type: Button, 
       props: { addFriend } 
      }]
    }
  }
}
```

可以看到上面代码中包含了四种不同的type属性，“button”，“div”，“p”和Button。当React看到一个element的type属性是一个类或者函数（如上面的`“type: Button”`），它将会用相应的props去确定该component会返回什么elements。

知道了这点，在这个过程的最后，React会拥有一个对象表示整个DOM树。在我们的例子中，它看起来是这样的：

``` js
{
  type: 'div', 
  props: {
    children: [{
      type: 'p',
      props: { children: 'Tyler McGinnis' }
    }, 
    { 
      type: 'button', 
      props: { 
        onClick: addFriend, 
        children: 'Add Friend'
      }
     }]
   } 
}
```

这个过程在React中叫做`reconciliation`，并且它会被`setState`和`ReactDOM.render`每一次调用触发。

现在，让我们回过头看看导致这篇博文的最开始的问题：

![](./react_elements_twitter.png)

这一刻，我们拥有了回答这个问题的全部知识，除了还有非常重要的一块内容。

问题在于无论你使用React多长时间，你不会使用`React.createElement`来创建DOM节点的对象表示。取而代之，你很可能使用JSX。

一开始我就写到：“产生困惑的首要原因是在JSX和React底层真正发生的存在一层常常未被触及的抽象层”。这个抽象层就是*JSX总是会被编译成`React.createElement`调用，通常是通过babel。*

让我们再看下前面的例子：

``` js
function Button ({ addFriend }) {
  return React.createElement(
    "button",
    { onClick: addFriend },
    "Add Friend" 
   )
}

function User({ name, addFriend }) { 
  return React.createElement(
    "div",
    null,
    React.createElement( "p", null, name),
    React.createElement(Button, { addFriend })
  )
}
```

这些代码就是JSX编译后的代码：

``` jsx
function Button ({ addFriend }) { 
  return ( 
    <button onClick={addFriend}>Add Friend</button> 
  )
}

function User ({ name, addFriend }) {
  return ( 
    <div>
     <p>{name}</p>
     <Button addFriend={addFriend}/>
    </div>
  )
}
```

那么最后，如果我们这么写我们的components：`<Icon/>`，那么我们如何称呼它？

我们称它为“创建一个element”，因为它就是编译后的JSX所做的事情。

```
React.createElement(Icon, null)
```

所有的这些例子，都是“创建一个React element”。

```
React.createElement(
  'div', 
  className: 'container', 
  'Hello!'
)

<div className='container'>Hello!</div> <Hello />
```

感谢阅读！关于这个主题的更多内容可以阅读Dan Abramov（译注：redux作者）的文章[React Components, Instances, and Elements](https://facebook.github.io/react/blog/2015/12/18/react-components-elements-and-instances.html)。

在Twitter上follow[作者Tyler McGinnis](https://twitter.com/tylermcginnis33)