---
title: p5js新手入门
date: 2017-07-30 21:47:12
tags:
  - p5js
  - 翻译
---

# p5js新手入门

p5js这个JavaScript库设计之初就与[Processing](https://processing.org/)有着共同的目标：让艺术家、设计师、教育工作者和初学者等都能够进行编码，并且是为现代web重新打造。它有着完整的一套作画功能，但并不意味着你能做的只是在canvas上画画。事实上，你可以把整个浏览器都当成你的“画布”，利用[插件库](https://p5js.org/libraries/)你可以很方便地与其他html5元素（如文本，输入框，视频，摄像头和音频）进行交互。

这儿有个p5js的[演示](http://hello.p5js.org/)。

接下来这篇文章会指导你搭建一个p5.js项目，以及第一次用它进行绘图。Processing的用户也许会想看下[Processing迁移教程](https://github.com/processing/p5.js/wiki/Processing-transition)。

## 下载及文件配置

最简单的开始方式是使用[p5.js完整下载包](https://p5js.org/download/)中附带的空例子。

如果你查看index.html文件，你会发现它链接到文件p5.js。如果你想用压缩后的（为了更快的加载速度）版本，就将链接地址改为p5.min.js。

```html
<script src="../p5.min.js"></script>
```

另外，你也可以使用在线托管的p5.js。p5.js的所有版本都部署到一个CDN（内容分发网络）上了。你可以在[p5.js CDN](https://cdnjs.com/libraries/p5.js)上查看它的历史版本。这时，你可以把链接地址改成：

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.5.11/p5.js"></script>
```

示例的HTML文件看起来可能会是这样：

```html
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.5.11/p5.js"></script>
    <script src="sketch.js"></script>
  </head>
  <body>
  </body>
</html>
```

## 环境

你可以使用任何你喜欢的[源码编辑器](https://en.wikipedia.org/wiki/Source_code_editor)。下面会包含[Sublime Text 2](http://www.sublimetext.com/2)的使用指南，其他好用的编辑器有[Brackets](http://brackets.io/)、[Atom](https://atom.io/)和[OpenProcessing](http://openprocessing.org/)。

打开Sublime，转到`File`菜单并且选择`Open...`，然后选择你的html和js所在的文件夹。在左边栏，你应当可以看到顶部显示了文件夹的名字，跟着显示了该文件夹包含的文件列表。

单击`sketch.js`文件，接着它会在右边的可编辑区域打开。

![](./sublime2.png)

为了查看画图的效果，你需要在浏览器打开`index.html`文件，这可以通过在文件管理器双击`index.html`文件，或者在浏览器地址栏输入`file:///the/file/path/to/your/html`达到。

## 首次绘画

在编辑器输入如下内容：

```js
function setup() {

}

function draw() {
  ellipse(50, 50, 80, 80);
}
```

这行代码的意思是“画一个中点距离左边50像素、距离顶部50像素，高和宽都是80像素的圆”。

保存你的`sketch.js`文件，然后刷新浏览器中的页面。如果你都输入准确了，你可以看到窗口显示如下：

![](./first-sketch.png)

如果你输入有错，那么你将看不到任何东西。如果这发生了，请先确保你准确地拷贝了例子的代码：数字应当包含在圆括号中并用逗号分隔，以及应当用分号作为行的结尾。

One of the most difficult things about getting started with programming is that you have to be very specific about the syntax. The browser isn't always smart enough to know what you mean, and can be quite fussy about the placement of punctuation. You'll get used to it with a little practice. Depending on the browser you are using, you can also see errors by looking at the JavaScript "console". In Chrome, for example, this is under View > Developer > JavaScript Console.

## 下一步
