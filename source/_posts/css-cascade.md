---
title: CSS层叠
date: 2016-12-05 23:39:48
tags:
  - css
  - 翻译
---

# 样式表层叠

> [原文地址](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade)

层叠是CSS的一个基本特征，它是一个定义了如何合并来自多个源的属性值的算法。它在CSS处于核心地位，CSS的全称层叠样式表正是强调了这一点。

## 哪些CSS实体会参与层叠计算

只有CSS声明，就是属性名值对，会参与层叠计算。这表示包含CSS声明以外实体的@规则不参与层叠计算，比如包含描述符（descriptors）的@font-face。对于这些情形，@规则是做为一个整体参与层叠计算，比如@font-face的层叠是由其描述符font-family确定的。如果对同一个描述符定义了多次 @font-face，则最适合的 @font-face是做为一个整体而被考虑的。

包含在大多数@规则的CSS声明是参与层叠计算的，比如包含于@media、@documents或者@supports的CSS声明，但是包含于@keyframes 的声明不参与计算，正如@font-face，它是作为一个整体参与层叠算法的筛选。

最后，注意 @import和@charset遵循特定的算法，并且不受层叠算法影响。

## CSS声明的源

CSS层叠算法期望通过挑选CSS声明来给CSS属性设置正确的值。CSS声明可以有不同的来源：

+ 浏览器会有一个基本的样式表来给任何网页设置默认样式。这些样式统称**用户代理样式**。一些浏览器通过使用真正的样式表，而其他则通过代码模拟，但无论是哪种情形都应是不可被检测的。而且部分浏览器允许用户修改用户代理样式。尽管HTML标准对用户代理样式做了诸多限制，浏览器仍大有可为，具体表现在不同浏览器间会存在重大的差异。为了减轻开发成本以及降低样式表运行所需的基本环境，网页开发者通常会使用一个CSS reset样式表，强制将常见的属性值转为确定状态。
+ 网页的作者可以定义文档的样式，这是最常见的样式表。大多数情况下此类型样式表会定义多个，它们构成网站的视觉和体验，即主题。
+ 读者，作为浏览器的用户，可以使用自定义样式表定制使用体验。

尽管CSS样式会来自这些不同的源，但它们的作用范围是重叠的，而层叠算法则定义了它们如何相互作用。

## 层叠顺序

层叠算法决定如何找出要应用到每个文档元素的每个属性上的值。

1. 它首先过滤来自不同源的全部规则，并保留要应用到指定元素上的那些规则。这意味着这些规则的选择器匹配指定元素，同时也是一个合适的@规则的一部分。
2.  其次，它依据重要性对这些规则进行排序。即是指，规则后面是否跟随者!import以及规则的来源。层叠是按升序排列的，这意味着来着用户自定义样式表的!important值比用户代理样式表的普通值优先级高：
    <table>
	<thead>
		<tr>
			<th></th>
			<th>来源</th>
			<th>重要程度</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>1</td>
			<td>用户代理</td>
			<td>普通</td>
		</tr>
		<tr>
			<td>2</td>
			<td>用户代理</td>
			<td>!important</td>
		</tr>
		<tr>
			<td>3</td>
			<td>用户</td>
			<td>普通</td>
		</tr>
		<tr>
			<td>4</td>
			<td>页面作者</td>
			<td>普通</td>
		</tr>
		<tr>
			<td>5</td>
			<td>CSS动画</td>
			<td>见下节</td>
		</tr>
		<tr>
			<td>6</td>
			<td>页面作者</td>
			<td>!important</td>
		</tr>
		<tr>
			<td>7</td>
			<td>用户</td>
			<td>!important</td>
		</tr>
	</tbody>
</table>
3.  假如层叠顺序相等，则使用哪个值取决于优先级。

## CSS动画与层叠

CSS动画，指使用@keyframes@规则定义状态间的动画。关键帧不参与层叠，意味着在任何时候CSS都是取单一的@keyframes的值而不会是某几个@keyframe的混合。

当有多个满足条件的关键帧时，在最重要的文档里面最后定义的关键帧会被选中，而不会是将它们组合在一起。

同时仍应注意用@keyframes @规则定义的值会覆盖全部普通值，但会被!important的值覆盖。

### 例子

用户代理CSS：

>     li { margin-left: 10px }

网页作者CSS1：

>     li { margin-left: 0 } /* This is a reset */

网页作者CSS2：

>     @media screen {
>      li { margin-left: 3px }
>     }
>
>     @media print {
>       li { margin-left: 1px }
>     }

用户CSS：

>     .specific { margin-left: 1em }

HTML：

>     <ul>
        <li class="specific">1<sup>st</sup></li>
        <li>2<sup>nd</sup></li>
       </ul>

对于这个情形，应当应用li和.specific里面的声明。由于没有声明被标记为!important，所以优先顺序为页面作者样式优于用户样式，用户代理样式最低。

故是这3条声明的竞争：

>     margin-left: 0

>     margin-left: 3px

>     margin-left: 1px

由于是在屏幕显示，所以最后一条舍弃，而前两条的选择器相同，因此优先级也相同，故最终选择的是后者：

>     margin-left: 3px

注意尽管定义在用户CSS里面的声明有更高的优先级，但它并不会被选中，因为层叠算法是先于优先级算法的。