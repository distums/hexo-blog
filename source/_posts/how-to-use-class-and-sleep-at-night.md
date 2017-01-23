---
title: 如何使用类而能够在晚上安然入睡
date: 2017-01-04 23:17:58
tags:
  - 翻译
  - js
---

# 如何使用类而能够在晚上安然入睡

> [原文地址](https://medium.com/@dan_abramov/how-to-use-classes-and-sleep-at-night-9af8de78ccb4#.8x6luasmc)

当前JS社区有个观点正在增长，那就是ES6的类不够[炫酷](https://github.com/joshburgess/not-awesome-es6-classes/)：

- 类模糊了JS基于原型继承的本质

- 类鼓励使用继承但你应该优先考虑使用组合

- 类会使你局限于最初想到的坏的设计

我认为js社区关注使用类和继承所带来的问题是很好的，但我也担忧初学者会对类一方面是“不好的”，一方面它又被加到语言中感到困惑。更令人困惑的是一些库，特别是React，ES6的类在它的文档中随处可见。难道React有意在跟随“坏的实践”？

我认为我们正处于这样一个怪异的过渡时期：对类的广泛使用是无可避免的恶，因为它们起了限制作用。这当然好过每学一个框架就要学习它自带的新的特殊类系统，而且它们每个都以mixin的形式各自实现了自己的多继承。

如果你喜欢函数式编程，你可能看到了从专有类系统转到“剥离的”ES6类（没有mixin，没有自动绑定，等等）如何使我们更进一步接近函数式解决方案。

同时，下列是如何使用类而能够在晚上安然入睡的建议：

- *限制使用类作为你的公共API*。（当然导出一个React Component是个例外因为它们并不是被直接使用。）你总是可以使用工厂函数来隐藏你的类。如果你暴露了它们，其他人会以各种对你来说讲不通的方式来继承它们，但你却有可能在未来破坏了继承类。避免破坏其他人的类是困难的，因为他们可能已经使用了你打算在未来版本使用的方法名字，或者他们可能使用了你的私有状态，或者他们可能在你的类实例里面嵌入了他们自己的状态，再有他们可能重写了你的方法但没有调用`super`，幸运的话这可能可以工作一段时间，但未来肯定会崩溃。这就是说，当基类和用户的继承类不存在来回往复地交互时，比如在React Component的情况下，导出一个基类作为API就是一个非常合适的选择。

- *不要继承超过一次*。继承作为一种捷径可以是方便的，而且继承一次是OK的，但不要继承更深。继承的问题在于派生类有太多权利访问继承层级上的每个基类的实现细节，反之亦然。当需求变更时，重构一个类层级结构会是如此之难，伴随着追溯过时的需求，它会变成一个烫手山芋。作为创建类层级的替代，考虑创建一些工厂函数。它们可以互相调用形成一条链，整合彼此的行为。你也可以要求“基”工厂函数接受一个“策略”对象来调整行为，并且用另一些工厂函数来提供这个对象。无论你选择何种方式，重要的部分是保持每一步的输入和输出是明确的。“你需要重新一个方法”不是一个明确的API，并且也很难设计好，但“你需要提供一个函数作为参数”就是明确的，并且能帮助你思考它。

- *不要在方法里面使用`super`调用*。如果你在派生类重写了一个方法，那么就完全重写它。追踪一连串的`super`调用就像调查一系列被反派藏在世界各地的记录。这仅仅是在看其他人干这个事的时候才显得有趣。假若你确实需要转化`super`调用的结果呢，或者是在做了其它什么操作之前或者之后转化呢？注意上一点：将你的类转换成一系列工厂函数，并且保持它们之间的关系明确。当你的所有工具仅仅是参数和返回值时，权衡职责会更加容易。处于中间的函数可以有与“顶层”和“底层”函数不同的接口，并且粒度更细。类并不容易提供类型的机制，因为你不能指定一个方法“仅仅供基类使用”或者“仅仅供派生类使用”，但是函数组合可以很自然地做到。

- *不要期望其他人使用你的类*。即使你选择提供类作为公开的API，也要优先考虑接受鸭子类型的输入。不是做`instanceof`类型检查，而是断言你要用的方法的存在性，并且相信用户做了正确的事。这将使用户拓展和后续调整更容易，同时也消除了在iframe里面和不同JavaScript执行环境里面的问题。

- *学习函数式编程*。它将帮助你不要以类的方式思考，这样你就不会不得不去使用它们尽管你知道它们的缺陷。

## 那么React Components怎样呢？

我希望上面的原则展示了为什么下面代码是个糟糕的主意。

``` js
import { Component } from 'react';
class BaseButton extends Component {
  componentDidMount() {
    console.log('hey');
  }
  render() {
    return <button>{this.getContent()}</button>;
  }
}
class RedButton extends BaseButton {
  componentDidMount() {
    super.componentDidMount();
    console.log('ho');
  }
  getContent() {
    return 'I am red';
  }
  render() {
    return (
      <div className='red'>
        {super.render()}
      </div>
    );
  }
}
```

但我认为下面的代码是可以接受的。

``` js
import { Component } from 'react';
class Button extends Component {
  componentDidMount() {
    console.log('hey');
  }
  render() {
    return (
      <div className={this.props.color}>
        <button>{this.props.children}</button>
      </div>
    );
  }
}
class RedButton extends Component {
  componentDidMount() {
    console.log('ho');
  }
  render() {
    return (
      <Button color='red'>
        I am red
      </Button>
    );
  }
}
```

是的，我们使用了令人敬畏的`class`关键字，但我们并没创建继承层级，因为我们始终从`Component`拓展。如果你希望，你甚至可以对此写一些lint规则。在上面的代码里面避免使用`class`是完全没必要的，这并不是什么大的问题。

当你期望提供一个组件并且能以通用的方式来拓展它时，[高阶组件](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750#.ab9g50dfu)几乎涵盖了我目前所碰到的每一种情形。本质上它们都是[高阶函数](https://en.wikipedia.org/wiki/Higher-order_function)。

在发掘出高阶组件之后，我从未觉得还需要`createClass()`式的mixins，ES7的mixins提案，[“stamp composition”](https://github.com/stampit-org/react-stampit/blob/master/docs/composition.md)，或者任何其他的组合式方案。这也是倾向仅使用`class`的另一个论点，因为你实际上并需要任何其他“更强大的”东西。

在React 0.14以后你可以使用[纯函数](https://github.com/reactjs/react-future/blob/master/01%20-%20Core/03%20-%20Stateless%20Functions.js)来写组件。这么做是完全值得的。任何时候你能用一个函数来替代一个类，你都该这么做。

然而，当你需要生命周期事件或者state时，除非React推出一些[纯函数式的](https://github.com/reactjs/react-future/tree/master/07%20-%20Returning%20State)[解决方案](https://github.com/reactjs/react-future/tree/master/09%20-%20Reduce%20State)，在不破坏上述规则的情况下使用类我认为并没有什么坏处。事实上从ES6类的方式迁移到纯函数式方案要比其他方式来得容易。

我也对使用不是直接建立在React的组合模型上的“组合式的方案”感到担忧，在我看来这是一种倒退，因为概念上它更接近于放在函数式范式中一点都不合理的mixins方案。

那么我对编写React组件的建议是什么呢？

- 你可以使用`class`如果你不继承两次以上并且不使用`super`。

- 尽可能优先考虑用纯函数来编写React组件。

- 在你需要state或者是生命周期时使用ES6的类。

- 在使用类的情况下，你只能直接从`React.Component`上继承。

- 关于函数式[状态](https://github.com/reactjs/react-future/tree/master/07%20-%20Returning%20State)[提案](https://github.com/reactjs/react-future/tree/master/09%20-%20Reduce%20State)向React团队反馈你的意见。

那么睡个好觉吧！