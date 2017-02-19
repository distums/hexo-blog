---
title: ES6类详解
date: 2017-02-08 22:27:23
tags:
  - js
  - 翻译
  - class 
---

# ES6类详解

[原文地址](https://ponyfoo.com/articles/es6-classes-in-depth)

## Javascript的类是什么意思？

Javascript是基于原型继承的语言，那么ES6类究竟又是什么？它是原型继承的语法糖——一种使语言对来自其他范式的程序员更有吸引力的手段，他们可能对原型链并不熟悉。ES6的很多特性（比如[解构](https://ponyfoo.com/articles/es6-destructuring-in-depth)）事实上都是语法糖，类也不例外。我乐意阐明这个事实因为这会使理解ES6类底层机制更加容易。对语言的巨大改造并不存在，只是使熟悉类的人更容易驾驭原型继承。

尽管我不喜欢把这特性称为“类”，我不得不承认这个语法事实上比用ES5的常规原型继承语法更易于使用，这也是每个人都能得到的好处——无论它是否叫做类或者其他名字。

让我们进入正题，我假定你已经理解了原型继承——因为正在阅读一篇关于Javascript的博客。下面你将这么编写一个类`Car`，它可以用于实例化，鼓起干劲然后行动吧。

``` js
function Car () {
  this.fuel = 0;
  this.distance = 0;
}

Car.prototype.move = function () {
  if (this.fuel < 1) {
    throw new RangeError('Fuel tank is depleted')
  }
  this.fuel--
  this.distance += 2
}

Car.prototype.addFuel = function () {
  if (this.fuel >= 60) {
    throw new RangeError('Fuel tank is full')
  }
  this.fuel++
}
```

为了是汽车前进，你将使用下面的代码。

``` js
var car = new Car()
car.addFuel()
car.move()
car.move()
// <- RangeError: 'Fuel tank is depleted'
```

漂亮！那么用ES6类又如何呢？它的语法非常接近声明一个对象，除了我们加上了前缀`class Name`，其中`Name`是要创建类的类名。这里我们使用昨天讲过的[method signature notation](https://ponyfoo.com/articles/es6-object-literal-features-in-depth#method-signatures)这个简洁的语法来声明类的方法。`constructor`和ES5的构造器函数类似，所以你可以用它来初始化任何你期望实例拥有的变量。

```js
class Car {
  constructor () {
    this.fuel = 0
    this.distance = 0
  }
  move () {
    if (this.fuel < 1) {
      throw new RangeError('Fuel tank is depleted')
    }
    this.fuel--
    this.distance += 2
  }
  addFuel () {
    if (this.fuel >= 60) {
      throw new RangeError('Fuel tank is full')
    }
    this.fuel++
  }
}
```

以防你未注意到，并且由于某些说不清的原因我确实疏忽它了，类的属性或者方法间的**逗号是不合法的**，与对象字面量语法的逗号（仍然）是必需的相反。这种差异一定会使正在尝试决定要使用一个对象或是一个类的人感到头疼，但是上面这些不带逗号的代码看起来确实整洁了些。

“类”通常会带有静态方法。以我们的老朋友`Array`为例，它除了拥有实例方法如`.filter`，`.reduce`和`.map`，还有静态方法，比如`Array.isArray`。在ES5的代码里，给我们的`Car`类添加这样的方法是相当容易的。

```js
function Car () {
  this.topSpeed = Math.random()
}

Car.isFaster = function (left, right) {
  return left.topSpeed > right.topSpeed
}
```

在ES6的`class`语法里，我们可以在方法前加前缀`static`，类似`get`与`get`的语法。同样的，这只是在ES5上的语法糖，因此很容易将它编译为ES5的语法。

``` js
class Car {
  constructor () {
    this.topSpeed = Math.random()
  }
  static isFaster (left, right) {
    return left.topSpeed > right.topSpeed
  }
}
```

ES6的`class`语法的一个优点在于你还可以使用`extends`关键字使从其他“类”“继承”变得更加容易。我们都知道消耗等量的汽油特斯拉汽车能开得更远，因此下面的代码展示了`Tesla extends Car`，并且“重写”（如果你曾经[使用过C#](https://msdn.microsoft.com/en-us/library/aa645768(v=vs.71).aspx)那么你很可能已经很熟悉这个概念了）了`move`以达到更远的距离。

```js
class Tesla extends Car {
  move () {
    super.move()
    this.distance += 4
  }
}
```

特殊的关键字`extends`指定了我们继承的类`Car`——用C#的话说，它类似与`base`。它存在的理由是大多数情况下我们可以通过在派生类重新实现来重写一个方法，在我们的`Tesla`的例子里，我们也应调用父类的方法。这样我们就不必要在重新实现方法的时候将代码拷贝到派生类。每当一个基类变更的时候都要将代码拷贝到每一个派生类是非常糟糕的，它会使维护我们的代码变成一个噩梦。

如果你做了如下操作，你将注意到由于`base.move()`特斯拉汽车移动了2个距离，这是每辆车都会做的，并且它还额外移动了4个距离因为**特斯拉**性能就是这么棒。

``` js
var car = new Tesla()
car.addFuel()
car.move()
console.log(car.distance)
// <- 6
```

你经常需要重写的方法是`constructor`。在里面你可以调用`super()`，传入基类需要的任何参数。特斯拉汽车的速度是两倍，因此我们就用翻倍的`speed`调用基类`Car`的构造函数。

```js
class Car {
  constructor (speed) {
    this.speed = speed
  }
}
class Tesla extends Car {
  constructor (speed) {
    super(speed * 2)
  }
}
```

明天，我们将讨论`let`、`const`和`for...of...`的语法。到时再见。