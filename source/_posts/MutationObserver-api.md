---
title: MutationObserver API
date: 2017-03-18 21:32:58
tags:
  - WebApi
  - 翻译
  - MutationObserver  
---

# MutationObserver API

[原文地址](https://davidwalsh.name/mutationobserver-api)

我最喜欢的网页技巧之一是使用CSS和JavaScript来检测DOM节点的插入和删除，祥见[Detect DOM Node Insertions with JavaScript and CSS Animations](https://davidwalsh.name/detect-node-insertion)。这个技巧和博文发布在我们没有适合的API来检测这样的事件的时期。现在我们有了[MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)，一个用于高效检测许多节点操作的API。让我们来看一下！

## 基本的MutationObserver API

对我来说`MutationObserver` API有点复杂，但下面是基本的设置：

```js
var observer = new MutationObserver(function(mutations) {
	// 为了观察，让我们将改变输出到控制台，看看这都是怎么工作的
	mutations.forEach(function(mutation) {
		console.log(mutation.type);
	});    
});
 
// 通知我全部信息
var observerConfig = {
	attributes: true, 
	childList: true, 
	characterData: true 
};
 
// 节点, config
// 在这种情况下我们将监听body及其子节点的全部改变
var targetNode = document.body;
observer.observe(targetNode, observerConfig);
```

为了使用`MutationObserver`需要有不少的代码，但可以分解为：

- 使用一个处理抛出的任何事件的callback创建`MutationObserver`的一个实例

- 为`MutationObserver`创建一套设置

- 调用`MutationObserver`实例的`observe`方法，传入要监听的节点（和它的子节点）以及设置信息

- 当你想停止检测的时候，调用`disconnect`

## MutationObserver设置

MDN提供了`MutationObserver`可设置项的详细信息：

- `childList`：如果需要检测目标节点的子节点（包含文本节点）的添加和移除则设置为true。

- `attributes`：如果需要检测目标节点属性的改变则设置为true。

- `characterData`：如果需要检测目标节点data值的改变则设置为true。

- `subtree`：如果要检测的不仅是目标节点，还包含其子孙节点则设置为true。

- `attributeOldValue`：如果`attributes`设置为true，并且需要记录目标节点属性改变前的值，则设为true。

- `characterDataOldValue`：如果`characterData`设置为true，并且需要记录目标节点data改变前的值，则设为true。

- `attributeFilter`：如果不需要检测全部属性的改变，则设置为属性localName（不包含命名空间）的一个数组。

这是相当多的当监听一个节点和/或其子节点需要注意的东西。

## MutationRecord：`MutationObserver`处理结果

当检测到一个改变时的结果对象也详细记录了：

- `type (String)`：如果是一个属性变化返回`attributes`，如果是一个CharacterData节点的变化返回`characterData`，如果是子节点的变化返回`childList`。

- `target (Node)`：根据type返回受改变影响的节点。对于`attributes`，返回属性改变的节点。对于`characterData`，返回CharacterData节点。对于`childList`，返回子节点改变了的那个节点。

- `addedNodes (NodeList)`：返回添加的节点。如果没有节点被添加了，则返回空的NodeList。

- `removedNodes (NodeList)`：返回删除的节点。如果没有节点被删除了，则返回空的NodeList。

- `previousSibling (Node)`：返回被添加或者删除节点的前一个兄弟节点，或者null。

- `nextSibling (Node)`：返回被添加或者删除节点的后一个兄弟节点，或者null。

- `attributeName (String)`：返回改变了的属性的localName，或者null。

- `attributeNamespace (String)`：返回改变了的属性的命名空间，或者null。

- `oldValue (String)`：返回值依赖于type。对于`attributes`，它是属性改变之前的值。对于`characterData`，它是节点data改变之前的值。对于`childList`，它是null。

好了。现在让我们看看`MutationObserver`的一些实际例子。

## 检测一个节点被添加了

我的[Detect DOM Node Insertions with JavaScript and CSS Animations](https://davidwalsh.name/detect-node-insertion)博文的使用场景是检测添加了新节点，因此让我们新建一个检测节点插入的代码段：

```js

// 让我们添加一个样例节点来看看MutationRecord是什么样的
// document.body.appendChild(document.createElement('li'));

{
	addedNodes: NodeList[1], // The added node is in this NodeList
	attributeName: null,
	attributeNamespace: null,
	nextSibling: null,
	oldValue: null,
	previousSibling: text,
	removedNodes: NodeList[0],
	target: body.document,
	type: "childList"
}
```

做为结果的MutationRecord显示了`addedNodes: NodeList[1]`，这意味着一个节点被添加到了DOM数的更深层次的某个位置。`type`的值是`childList`。

## 检测一个节点被移除了

移除一个节点显示了如下的MutationRecord：

```js
// 现在让我们查看一个节点被移除时的MutationRecord
// document.body.removeChild(document.querySelector('div'))

{
	addedNodes: NodeList[0],
	attributeName: null,
	attributeNamespace: null,
	nextSibling: text,
	oldValue: null,
	previousSibling: null,
	removedNodes: NodeList[1], // The removed node is in this NodeList
	target: body.document,
	type: "childList"
}
```

这个操作的`type`值同样是`childList`，但是现在`removeNodes`显示了`NodeList[1]`，一个包含被移除节点的`NodeList`。

## 检测属性变化

如果一个元素的属性改变了，你将马上就知道。MutationRecord会显示：

```js
// 属性变化了MutationRecord看起来会是怎样？
// document.body.setAttribute('id', 'booooody');

{
	addedNodes: NodeList[0],
	attributeName: "id",
	attributeNamespace: null,
	nextSibling: null,
	oldValue: null,
	previousSibling: null,
	removedNodes: NodeList[0],
	target: body#booooody.document,
	type: "attributes"
}
```

同样注意`target`会指示是那个节点的属性改变了。`oldValue`值会显示属性的前一个值，而标准的`getAttribute`检查会给你新的属性值。

## 停止监听

如果你希望编写最高效的app，你应仅仅在你需要监听的那段时间添加监听器，并且在结束后立马移除：

```js
observer.disconnect();
```

`MutationObserver`的实例拥有一个`disconnect`方法用于停止监听。因为你的app可能会有非常非常多的DOM操作，你会希望拥有能力在用户与页面交互的时候停止监听。

`MutationObserver` API似乎有点繁琐了，但是它是相当强大的，并且提供了丰富的信息，而且是无hack的。Daniel Buchner原创的高明的“hack”提供了检测节点添加和移除兼容性更好的方式，但是如果可能应使用`MutationObserver`。