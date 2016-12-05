---
title: things-you-may-not-have-known-about-node-js-modules
date: 2016-12-04 21:54:19
tags:
  - 翻译
  - nodejs
---

# 你未必知道的Node.js模块知识

> [原文地址](https://medium.com/@trstringer/things-you-may-not-have-known-about-node-js-modules-6ef926e4f4ff#.x243o93tj)

我们常常是一头扎入某样‘东西’，开始使用它却往往对它的一些优美之处缺乏真正的理解。Node.js的模块系统就是一个这样的‘东西’，我们认为仅仅需要知道使用`require()`，而轻率的忽略了其它的一些闪光之处。以下罗列一些你一开始未必知道的模块知识...

## 你可以检测模块是被直接运行还是通过require引用的

这儿有Python开发吗？在Python经常可以看到（或者使用）一个简单的条件测试`__name__ == '__main__'`来判断模块是被直接运行的或者是被引用的。Node.js也有相似的特性，只是它的条件测试看起来是这样的`require.main === module`。以下是一个简单的例子：

``` js
// 1.js
if (require.main === module) {
  console.log('1.js was run directly!');
}
else {
  console.log('1.js was require()\'d');
}
const two = require('./2');
// 2.js
if (require.main === module) {
  console.log('2.js was run directly!');
}
else {
  console.log('2.js was require()\'d');
}
```

运行`node 1`（你不需要包含文件拓展名，原因见下节）将得到如下输出：

> 1.js was run directly!
> 2.js was require()’d

这个功能在什么时候会有用呢？假设你在编写一个能够直接运行的模块（例如：`node your-module`），它拥有极好的功能且可能被其他模块引用了。这时你就可以用这个条件测试来决定是否在前一种情形下触发某些行为。

## 你不必指定'.js'（或者'.json'）拓展名

Node.js并不要求你在加载模块时指定文件的拓展名假设目标文件拥有如下拓展名：`js`、`json`或者`node`（*.node表示二进制插件）。这是完全没问题的，我们不需要指定`require('./2.js')`而可以做一些简化：

``` js
// 1.js
const two = require('./2');
console.log(two.message);
// 2.js
exports.message = "hi from 2.js!";
```

以上代码会如预期输出消息`hi from 2.js!`。但是如果有多个文件名相同但是拓展名不同的模块时会发生什么？换句话说，存在与`1.js`同级的且内容如下的文件`2.json`时：

``` js
{
  "message": "hi from 2.json!"
}
```

那么`require('./2')`调用会先查找`2.js`文件。如果找到了就使用这个文件，否则它就会尝试去查找文件`2.json`。

## 你可以加载一个目录

我注意到这点是被广泛采用的。事实上，Node.js对此完全支持，只要该目录要么显式在`package.json`文件的`main`属性指定要加载的文件，要么包含一个index.\<ext\>（`etc`为默认支持加载的三个拓展名`js`、`json`或者`node`之一）文件。以下是一个例子：

``` js
// 1.js
const dirIndex = require('./mydir');
// mydir/index.js
console.log('hello from mydir/index.js!');
```

运行`node 1`会如预期输出`hello from mydir/index.js!`。这是因为我们加载了一个目录，node.js在发现该目录存在一个`index.js`然后加载了它。

这在使用者不需要知道或者根本不关注目录结构时提供了极大的便利以及代码可读性，目录本身就能作为某种形式上的代码容器。

## 引用过的模块会被缓存

猜下如下代码会输出什么？

``` js
// 1.js
const two = require('./2');
console.log(two.myDateTime);
setTimeout(() => {
  const twoAgain = require('./2');
  console.log(twoAgain.myDateTime);
}, 2000);
// 2.js
exports.myDateTime = Date().toLocaleString();
```

如果你推测输出的日期/时间字符串是相同的，那么恭喜你你是正确的！为什么会这样？node.js应该加载了那个模块两次，2秒间隔前后各一次，对吗？不！原因是node.js会缓存已经加载过的模块，正如我们所看到的。`Date().toLocaleString()`只会执行一次，然后node.js就将它缓存了并给`two.myDateTime`赋值，同样的值也赋给了接下来的`twoAgain.myDateTime`。

## exports只是module.exports的引用

我把这条保留到了最后是因为我认为它是一堆困惑的根源。我发现遍地都是这样的疑问：“什么时候该使用`exports`，什么时候使用`module.exports`？”。然而一旦你发现`exports`不过是`module.exports`的引用这个简单的事实，你就不需要再去刻意记住什么或者再去琢磨它。

``` js
console.log(`exports is module.exports is a ${exports === module.exports} statement`);
```

运行它，你将得到一条事实：`exports is module.exports is a true statement`。还不足够说服你吗？

``` js
exports.message = 'hello world';
console.log(`exports.message is ${exports.message}`);
console.log(`module.exports.message is ${module.exports.message}`);
module.exports.anotherMessage = 'hey universe';
console.log(`exports.anotherMessage is ${exports.anotherMessage}`);
console.log(`module.exports.anotherMessage is ${module.exports.anotherMessage}`);
```

请注意了，你将看到如下输入：

> exports.message is hello world
> module.exports.message is hello world
> exports.anotherMessage is hey universe
> module.exports.anotherMessage is hey universe

但是，这儿有个陷阱。由于`exports`仅仅是`module.exports`的引用，若是你做了`exports = ...`你将碰到预期外的行为。此刻，`exports`不再指向`module.exports`了，而这也肯定不是你想要的结果。

## 总结

掩盖看似简单的东西是如此容易及令人舒适。但是，花费时间去调查即使是最简单的事物的每个角落也是值得的，你会发现它们拥有一些你不曾知道的不可思议的且炫酷的特性！希望我的这篇博文阐明了Node.js模块系统的其中一些特性。