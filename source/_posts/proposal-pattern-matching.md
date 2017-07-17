---
title: JavaScript模式匹配提案
date: 2017-07-15 15:13:15
tags:
  - 翻译
  - js
  - proposal
---

> 本文翻译自[proposal-pattern-matching](https://github.com/tc39/proposal-pattern-matching)，同时会加上一些[这篇文章](https://ponyfoo.com/articles/pattern-matching-in-ecmascript)中的例子来帮助理解。

# ECMAScript模式匹配语法

Stage 0提案
提案者: Brian Terlson (Microsoft, [@bterlson](https://twitter.com/bterlson)), Sebastian Markbåge (Facebook, [@sebmarkbage](https://twitter.com/sebmarkbage))

```js
let getLength = vector => match (vector) {
    { x, y, z }: Math.sqrt(x ** 2 + y ** 2 + z ** 2),
    { x, y }:    Math.sqrt(x ** 2 + y ** 2),
    [...]:       vector.length,
    else: {
        throw new Error("Unknown vector type");
    }
}
```

模式匹配是基于数据的结构来选择不同行为的手段之一，其方式类似于解构。比如，你可以毫不费力地用指定的属性来匹配对象并且将这些属性的值绑定到匹配分支上。模式匹配使非常简洁和高度可读的函数式模式成为可能，并且已存在于许多语言之中。这个提案从[Rust](https://doc.rust-lang.org/1.6.0/book/patterns.html)和[F#](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/pattern-matching)汲取了许多灵感。

本提案当前处于stage 0阶段，因此不排除会有重大的变更。任何反馈和意见都不胜感激。请使用issue来提交问题或者想法，以及发送pull request来更新内容。修订、澄清，尤其是使用示例都将十分有帮助。

## 语法概览

```
Expression :
  MatchExpression
  
MatchExpression :
  `match` [no |LineTerminator| here] `(` Expression `)` [no |LineTerminator| here] `{` MatchExpressionClauses `}`
  // 备注：这里需要一个cover grammar来处理一个match函数调用和match表达式间的歧义

MatchExpressionClauses : 
  MatchExpressionClause
  MatchExpressionsClauses `,` MatchExpressionsClause
  
  // MatchExpressionClauses按顺序执行直到有一条返回了真值
  
MatchExpressionClause :
  MatchExpressionPattern `:` AssignmentExpression
  
MatchExpressionPattern :
  ObjectMatchPattern
  ArrayMatchPattern
  IdentifierMatchPattern
  LiteralMatchPattern
  `else`

  
ObjectMatchPattern :
  // 基本ObjectBindingPattern，以及可选的剩余元素绑定
  
ArrayMatchPattern :
  // 基本ArrayBindingPattern，以及可选的剩余元素绑定
  
IdentifierMatchPattern :
  // 任何绑定标识符

LiteralMatchPattern :
  // number, string, boolean, null, undefined 字面量
```

对象和数组模式的语法有意地设计成与解构保持一致，因为出于以下理由这是有好处的。首先，它与开发者已经熟悉的现有语法相一致。其次，它允许在类似的场景中使用模式匹配和解构（例如，将来给`multi-methods`<sup><a href="#multi-methods">[1]</a></sup>等的提案）。然而在实际中，模式匹配JavaScript数值需要比简单的解构更具表达能力。这份提案添加了额外的模式来填补空白。为了提升本提案的实用性和表达能力，与解构更进一步分离也许也是合理的（比如，类似于[#17](https://github.com/tc39/proposal-pattern-matching/issues/17)那样）。

## 对象模式

对象模式用指定的属性来匹配对象，匹配的对象上可以包含额外的属性。例子：

```js
match (obj) {
    { x }: /* 匹配带有属性x的对象 */,
    { x, ... y }: /* 匹配带有属性x的对象，任何额外的属性填充到y中 */,
    { x: [] }: /* 匹配x属性是一个空数组的对象 */,
    { x: 0, y: 0 }: /* 匹配x和y属性值为0的对象 */
}
```

```js
const matchPoint = point => match (point) {
  { x, y }: [x, y]
}
matchPoint({ x: 5, y: 7 })  // <- [5, 7]
matchPoint({ x: 5, y: 7, z: 9 })   // <- [5, 7]
matchPoint({ x: 3, z: 7 })  // <- Error

const matchPointWithElse = point => match (point) {
  { x, y }: [x, y],
  else: [0, 0]
}
matchPointWithElse({ x: 3, z: 7 })  // <- [0, 0]

const matchNullPoint = point => match (point) {
  { x: 0, y: 0 }: [x, y]
}
matchNullPoint({ x: 0, y: 0 })  // <- [0, 0]
matchNullPoint({ x: 1, y: 1 })  // <- Error

const isUSD = item => match (item) {
  { options: { currency: 'USD' } }: true,
  else: false
}
isUSD({ value: 19.99, options: { currency: 'USD' } })   // <- true
isUSD({ value: 19.99, options: { currency: 'ARS' } })   // <- false
```

## 数组模式

数组模式匹配类数组对象（拥有`length`属性的对象）。`...`绑定（可以是匿名的）用于匹配任意长度的数组。例子：

```js
match (arr) {
    []: /* 匹配空数组 */,
    [...]: /*匹配任何数组 */,
    [x]: /* 匹配长度为1的数组，其第一个元素绑定为x */,
    [ x, ... ]: /* 匹配长度至少为1的数组，其第一个元素绑定为x */,
    [ { x: 0, y: 0 }, ... ]: /* 匹配第一个元素是二维坐标原点的数组 */
}
```

也可以让数组模式支持可迭代对象，然而这个设计是否可取尚不明确。首先，模式匹配的用户是不是期望数组模式去匹配任何实现了`Symbol.iterable`的对象还不清楚。其次，由于遍历一个可迭代对象是有副作用的，因此围绕在每个匹配分支上可迭代对象所处状态会引起诸多困惑，而且也不清楚默认有副作用的模式匹配是不是个好主意。

尽管如此，解构确实是可以工作在可迭代对象上的，因此这在一致性方面尚有争议。

## 字面量模式

字面量模式是string型，number型，bool型，null以及undefined的字面量，并且精确匹配该值。例子：

```js
match (val) {
    1: /* 匹配Number型值1 */,
    "hello": /* 匹配String型值"hello" */,
}
```

## 标识符模式与Symbol.matches

标识符会查找它们运行时候的值。一个数值是匹配的只要它实现了`Symbol.matches`方法，并且该方法在输入数值是匹配的时候返回了真值（另外解构`Symbol.matches`返回值的一种方法请参考下面*可选拓展*部分）。

这个功能带来了一些优点。第一，它允许匹配正则表达式。当然也可以考虑加个RegExp模式，但是正则表达式（特别是很复杂的那些）通常不会用‘inline’的方式声明。

第二，它允许简单的类型/instanceof检查——一个类型可以实现它自己的`Symbol.matches`方法用于决定某个值是不是该类型。一个简单的实现可以仅仅是`return value instanceof this.constructor`。如此简单的实现可以在通过`class`关键字新建类型的时候默认添加上。

第三，更加一般地，它围绕数值间的互相匹配创建了一个协议。这在未来的提案中也许会很有用，比如`interface`提案，用于添加类似于`nominal interface`<sup><a href="#nominal-interface">[2]</a></sup>或者`tagged union discrimination`<sup><a href="#nominal-interface">[2]</a></sup>的东西。

```js
match (val) {
    someRegExp: /* val匹配正则表达式 */,
    Array: /* val是数组的实例 */,
    CustomType: /* val是CustomType的实例 */,
    PointInterface: /* 也许是个tagged union，或者其他类似的 */
}
```

```js
// 正则表达式匹配
const numbers = /^-?\d+,\s*-?\d+$/
const matchPoint = point => match (point) {
  numbers: point.split(/,\s*/).map(n => parseInt(n))
}
matchPoint(`7, -3`) // <- [7, -3]

// 匹配实现Symbol.matches的对象
const threeDigitNumber = {
  [Symbol.matches](value) {
    return value >= 100 && value < 1000
  }
}
const matchPointWithSymbol = point => match (point) {
  threeDigitNumber: point.toString.split(``).map(n => parseInt(n))
}
matchPointWithSymbol(735) // <- [7, 3, 5]

// 运行时的类型检查，这里Number需要实现Symbol.matches
const matchPointWithTypeCheck = point => match (point) {
  { x: Number, y: Number }: [x, y]
}
matchPointWithTypeCheck({ x: 1, y: 2 }) // <- [1, 2]
matchPointWithTypeCheck({ x: 1, z: 2 }) // <- Error
matchPointWithTypeCheck({ x: 1, y: 'two' }) // <- Error
```

## 更多的例子

### 嵌套模式

模式可以嵌套。例如：

```js
let isVerbose = config => match (config) {
    { output: { verbose: true } }: true,
    else: false
}
```

上述模式中的`true`也可以是任何其他模式（在这个例子中它是字面量模式）。

### 匹配嵌套

由于match是一个表达式，你可以在一个match分支的后项中更进一步匹配。考虑：

```js
let node = {
    name: 'If',
    alternate: { name: 'Statement', value: ... },
    consequent: { name: 'Statement', value: ... }
};

match (node) {
    { name: 'If', alternate }: // if with no else
        match (alternate) {
            // ...
        },
    { name: 'If', consequent }: // if with an else
        match(consequent) {
            // ...
        }
}
```

## 设计目标与替代方案

### 没有fall-through

`Fall-through`<sup><a href="#fall-through">[3]</a></sup>可以通过`continue`关键字实现。如果没有匹配的模式，这将会是一个运行时的错误。

### 声明vs表达式

将`match`设计为声明将会使它看起来与`switch`子句非常一致。然而与`switch`一致可能会是有问题的，因为分支的行为会表现地不同。使用`switch`做为`match`的思维模型是有帮助的，但并不能涵盖全部。

同时也没有足够的理由要让这个语法只支持声明。解析上下文或者仅限声明的`match`所存在的困难会限制其实用性。另一方面，表达式形式的`match`可以很方便地在各个地方使用，尤其是做为箭头函数的函数体。

### 匹配分支语法

对于匹配体的语法存在很多选择，大体上说有：类case语法，类箭头函数语法，以及只包含表达式的语法。

#### 类Case语法的分支

类Case语法的分支包含声明。这种分支的后项按一个声明接一个声明的顺序执行直到遇到流程控制关键字（或者达到case结构的末尾）。类Case的分支非常有用因为它们允许声明做为子元素，`throw`声明就经常会用到。

类Case的分支在语句构成上比较难处理，因为你需要一个关键字来表示一个分支的开始。一个显而易见的选择是使用`case`。找到其他符合语境的关键字会比较困难，但也许也不是完全不可能。

另外，由于`match`表达式的值是第一个匹配分支执行后的值，模式匹配的用户将不得不理解语义上不是JS开发者通常所认为的那样的`completion value`<sup><a href="#completion-value">[4]</a></sup>。

最后，类case的分支在更小的场景中使用会让模式匹配显得繁琐。

#### 类箭头函数语法的分支

箭头函数支持一个表达式或者一个可选的语句块。把这应用到我们的模式匹配的语法带来了两个很好的特性：简洁而不失可拓展性，和用`,`分隔的分支。这个方案应用到了上述所有例子中。

#### 仅表达式语法的分支

你也可以只允许声明出现在一个分支中，然后依赖于`do`表达式来提供声明。不过这看起来有点不如类箭头函数分支友好。

### Else分支语法

我这里没有过多介绍`else`是因为它与JavaScript的其他部分是一致的，但你也许会更喜欢类似`_`这种更紧凑的语法（尤其是如果你用过F#）。`_`还有可以绑定到一个值的好处，这可以保证你以无副作用的方式引用一个值。考虑：

```js
let obj = {
    get x() { /* 计算很多东西 */ }
};

match (obj.x) {
    //...
    else: obj.x // 重新计算
}

match (obj.x) {
    // ...
    _: _ // 对obj.x求值的结果会绑定为_并且返回
}
```

## 可选拓展

### 对象和数组模式匹配数值

数组模式可以拓展成允许带一个数值以用任何二元操作符将属性或者元素和某个特定值进行比较。例如：

```js
// point对象的x和y属性不能大于100
let isPointOutOfBounds = p => match (p) {
    { x > 100, y }: true,
    { x, y > 100 }: true,
    else: false
}
```

### If判断

可以在匹配分支上下文中对待匹配值做各种测试往往是便利的。例如：

```js
match (p) {
    { x, y } if x === y: true,
    else: false
};
```

### 解构运行时的匹配结果

支持解构运行时的匹配结果可能是很有用的，尤其是正则表达式的匹配。考虑：

```js
let nums = /(\d)(\d)(\d)/;
let lets = /(\w)(\w)(\w)/;
let str = '123';
match (str) {
    nums -> [, first, second, third]: first + second + third,
    lets -> [, first, second, third]: first + second + third
}
```

正则表达式对象的`Symbol.matches`方法会被调用，并且如果匹配成功了，则返回match对象。这个match对象可以通过`->`字句更进一步解构。

### 匹配多个模式

有些时候匹配多个模式是有用的。这可以通过使用`||`来分割多个模式（类似于Rust和F#）。你也可以通过`&&`来要求一个值匹配多个模式。

### 对象默认‘严格匹配’

在上文提案中，对象上的额外属性是允许的，而更长的数组则是不允许的除非你显示使用`...`来匹配。这同样也可以应用到对象上：`{x}`将仅匹配只拥有一个名为x的属性的对象，而`{x,...}`匹配任何拥有属性x的对象。但是，对象匹配的主要使用场景很可能并不关心额外的属性，而且通过`_`属性或者Symbol键值来扩充对象以添加额外的metadata是很普遍的，提案的语法看起来是没问题的（当然linter是可以将这强制为显示的如果它们希望）。

### 数组模式匹配可迭代对象

在上文提案中，数组模式仅对拥有`length`属性的类数组对象有效。它可以拓展为支持任何可迭代对象，但必须注意避免产生副作用以及在匹配分支间移动时多次迭代可迭代对象。

### 没有围绕匹配数值的圆括号

（我认为）`cover grammar`<sup><a href="#cover-grammar">[5]</a></sup>是可以避免的，通过进一步摆脱`switch`的语法同时省略`match`数值旁边的圆括号：

```js
match val {
    // ...
}
```

这移除了和对一个名为`match`函数的调用之间的歧义。

### 内置Symbol.matches实现

`Symbol.matches`可以在许多内置类型上实现，比如Number和String，用于匹配该类型的数值。另外，类可以创建`Symbol.matches`方法来为你做`instanceof`检查。

## 译注

1. <a name="multi-methods"></a>**multi-methods**：[多分派](https://zh.wikipedia.org/wiki/%E5%A4%9A%E5%88%86%E6%B4%BE)，可以到[这个库](https://github.com/KrisJordan/multimethod-js)感受下；

2. <a name="nominal-interface"></a>**nominal-interface**和**tagged union discrimination**：

3. <a name="fall-through"></a>fall-through

4. <a name="completion-value"></a>completion-value

5. <a name="cover-grammar"></a>cover-grammar
