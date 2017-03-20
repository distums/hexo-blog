---
title: ECMAScript的Observable提案
date: 2017-03-19 21:26:49
tags:
  - js
  - 翻译
  - observable
---

# ECMAScript的Observable提案

[原文地址](https://ponyfoo.com/articles/observables-coming-to-ecmascript?utm_source=tuicool&utm_medium=referral)

在JavaScript，Observables很大程度上是通过诸如[RxJS](https://github.com/Reactive-Extensions/RxJS)和[Bacon.js](https://baconjs.github.io/)这样的库流行起来的。JafarHusain，一个Netflix的技术总监，函数式编程的长期拥护者，同时也是TC39的成员，制定了一份提案用于在js语言核心功能中引入Observables。在编写本文的时候，[Observables提案](https://github.com/tc39/proposal-observable)处于stage1状态，但已被标记为准备移到stage2状态。

## Observable和observer API

在提案里，`Observable`成为了一个新的内置对象，用于处理事件流。`Observable`构造函数接收一个用于定于事件流的回调函数。在下面的例子里，我们的observable返回了一个仅包含值`1`和`2`的事件流。`observable.next`方法可以用于往观察流里面添加事件。

```js
new Observable(observer => {
  observer.next(1)
  observer.next(2)
})
```

我们可以使用`observer.error`来报告流处理过程中发生的错误。

```js
new Observable(observer => {
  observer.error(new Error(`Failed to stream events`))
})
```

我们可以使用`observer.complete`来标识一个流已经处理结束。

```js
new Observable(observer => {
  observer.next(1)
  observer.next(2)
  observer.complete()
})
```

传入`Observable`构造函数的回调可以返回一个清理函数来释放我们的可观察对象。这在需要移除事件监听器、清除timeout和类似的清理工作时会很有用。下面的这个可观察对象比上面那个来得有趣。它跟踪用户在屏幕上移动光标时鼠标相对于页面的位置，然后在这一过程中产生一个描述光标在页面位置的事件流。

```js
function mouseTracking () {
  return new Observable(observer => {
    const handler = ({ pageX, pageY }) => {
      observer.next({ x: pageX, y: pageY })
    }

    document.body.addEventListener(`mousemove`, handler)

    return () =>  {
      document.body.removeEventListener(`mousemove`, handler)
    }
  })
}
```

为了订阅一个可观察事件流，我们仅需调用可观察对象实例的`Observable#subscribe`方法。这么做将调用上例代码中传入`Observable`构造函数的回调函数，绑定事件监听器并且开始事件流。移动鼠标现在会引起事件被触发进事件流。

```js
mouseTracking().subscribe({
  next({ x, y }) { console.log(`New position: ${ x }, ${ y }`) },
  error(err) { console.log(`Error: ${ err }`) },
  complete() { console.log(`Done!`) }
})
```

## Subscription#unsubscribe

`Observable#subscribe`返回一个可以让我们`unsubscribe`的对象，同时如果存在清理函数则执行它。当我们不在关注可观察流的事件时，我们可以取消订阅，并且让可观察对象清理自身。

```js
const subscription = mouseTracking().subscribe({
  next({ x, y }) { console.log(`New position: ${ x }, ${ y }`) },
  error(err) { console.log(`Error: ${ err }`) },
  complete() { console.log(`Done!`) }
})

subscription.unsubscribe()
```

## Observable.of

`Observable.of(...items)`是一个简单的静态工具函数，用于从提供的`items`新建一个`Observable`对象。然后当`Observable#subscribe`调用的时候会同步分发这些`items`。

```js
Observable.of(1, 2, 3, 4).subscribe({
  next(item) { console.log(item) }
})
// <- 1
// <- 2
// <- 3
// <- 4
```

我们可以认为如下的代码是`Observable.of`的简化实现：对于给定的值我们返回一个同步流。

```js
Observable.of = (...items) => {
  return new Observable(observer => {
    items.forEach(item => {
      observer.next(item)
    })
    observer.complete()
  })
}
```

## Observable.from

这个静态方法将给定参数转为一个`Observable`对象。如果给定的`Object`拥有一个`Symbol.observable`，则返回调用该方法的结果。

```js
Observable
  .from({
    [Symbol.observable]() { return Observable.of(1, 2, 3) }
  })
  .subscribe({
    next(item) { console.log(item) }
  })
// <- 1
// <- 2
// <- 3
```

如果给定参数没有实现`Symbol.observable`方法，则假定该对象是可遍历的。这将返回该可遍历对象的一个同步`Observable`序列。

```js
Observable
  .from([1, 2, 3])
  .subscribe({
    next(item) { console.log(item) }
  })
// <- 1
// <- 2
// <- 3
```

在这种情形下，`Observable.from`与`Observable.of`类似。我们可以认为如下是`Observable.from`的简化实现。

```js
Observable.from = value => {
  if (typeof value[Symbol.observable] === `function`) {
    return value[Symbol.observable]()
  }
  return Observable.of(Array.from(value))
}
```

## 总结

注意这个提案仍然处于早期阶段，但是它将为原生JavaScript级别的函数式编程打下基础。最终，它也许会提供`Observable#filter`或者`Observable#map`事件流的能力，允许我们只关注我们想要监听的那一类型事件。

同时，随着我们持续关注这一模式，让规范自然而然地逐渐演化，用户群体也可以选择自己实现它。你可以在GitHub仓库上找到当前版本规范的一个[polyfill](https://github.com/tc39/proposal-observable/blob/0fa13995f372bab50de8cb5e8db59066ad08dd7a/src/Observable.js)，但是如果你想在浏览器的DevTools里把玩一下你得删掉`export`关键字。