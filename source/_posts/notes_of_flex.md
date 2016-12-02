---
title: Notes of flex
date: 2016-12-03 03:13:03
tags: 
  - css 
  - flex
---

# Of container

1. display:flex生成块级flex box,display:inline-flex生成行内flex box

2. container的flex-direction建立main axis,justify-content定义item在main axis的排列方式,align-items定义item在cross axis的排列方式,item的align-self可以重写此方式

3. container的flex-direction和direction决定item的排列方向

4. container设置flex-wrap:wrap可以在所需宽度不足时多行显示

5. 直接包含在container里面的文本会包裹在匿名的flex item里（可否设置flex值,*选择符貌似无法选中这个匿名item）

   > Text directly contained in a flex container is wrapped in an anonymous flex item

6. Everything outside a Flex Container and inside a Flex Item is rendered as usual

# Of item

1. item设置flex:0则宽度根据内容计算,不设置flex相当于flex:0

2. item的flex是flex-grow,flex-shrink和flex-basis的缩写

3. item的order可以设置排列顺序,不设置order值等同于0

4. item设置为absolute定位,会相对于container的content-box corner定位

# Questions

1. item的width/height和main-size/cross-size的关系

2. align-content/align-items

3. flex+margin