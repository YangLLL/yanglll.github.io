---
title: flex 布局
date: 2021-08-20 16:53:02
tags:
---

### flex布局
flexbox是一种一维布局模型（Grid是二维布局），flex布局的作用是设置容器内的项目怎么排列，分配空间。  
flex布局有两条轴线，主轴和交叉轴，主轴由 flex-direction 定义，交叉轴和主轴垂直。
容器通过设置 `display: flex | inline-flex` 来使用 flex 布局

flex布局的常用属性：
- flex
- flex-direction
- flex-wrap
- flex-flow
- justify-content
- align-items
- align-content

#### flex
flex的作用是设置容器内的项目，如何缩放来适应容器内的可用空间  
flex是 `flex-grow`，`flex-shrink`，`flex-basis` 的简写，flex元素是在 flex-basis 的基础上缩放的。

```
flex: <flex-grow> <flex-shrink> <flex-basis>
```
flex取值：
- initial: 相当于 `flex: 0 1 auto`，不会拉伸，可以收缩防止溢出  
- auto: 相当于 `flex: 1 1 auto`，flex 元素在需要的时候既可以拉伸也可以收缩  
- none: 相当于 `flex: 0 0  auto`，不可伸缩  
- 一个数字`<number>`：相当于 `flex: <number> 1 0`，`<flex-shrink>` 的值被假定为1，`<flex-basis>` 的值被假定为 0 。
`flex: 0`，相当于 `flex: 0 1 0`,  

#### flex-grow
设置 flex 增长系数，处理 flex 元素在主轴上增加空间的问题。  
如果所有的兄弟元素都有相同的 flex-grow 系数，那么所有的项目平分剩余空间，如果值不同，根据不同的 flex-grow 系数定义的比例来进行分配  

取值：
- 数字`<number>`：正数，负值无效，默认为0。设置为 0 不会超过 flex-basis 的尺寸

#### flex-shrink
设置 flex 元素的收缩规则，处理 flex 元素在主轴上收缩的问题。  
flex-shrink 只有在 flex 元素总和超出主轴的时候才会生效 

取值：
- 数字`<number>`：正数，不能设置负值。值越大，收缩的比例越大。

#### flex-basis
指定了 flex 元素在主轴方向上的初始大小，在这个初始值的基础上进行收缩  
取值：
- <'width'>：可以是固定值（平常width可取的值都可以，比如max-content），也可以是相对于父弹性盒容器主轴尺寸的百分数。默认为 auto，也就是元素内容的尺寸大小  
- content：基于 flex 的元素的内容自动调整大小

#### flex-directoin：设置主轴的方向  
取值：
- row：水平排列  
- row-reverse：水平排列，从右边开始  
- column：垂直排列  
- colum-reverse：垂直排列，从下往上  

#### flex-wrap：是否换行
取值：
- wrap：换行
- nowrap: 不换行（默认）
- wrap-reverse：换行，第一行在下面

#### justify-content：沿主轴的对齐方式
取值：
- flex-start：从行首开始排列，每行第一个弹性元素与行首对齐，同时所有后续的弹性元素与前一个对齐  
- flex-end：从行尾开始排列，每行最后一个弹性元素与行尾对齐，其他元素将与后一个对齐  
- center：元素向每行中点排列，每行第一个元素到行首的距离将与每行最后一个元素到行尾的距离相同。  
- space-between：在每行上均匀分配弹性元素。相邻元素间距离相同。每行第一个元素到行首的距离和每行最后一个元素到行尾的距离将会是相邻元素之间距离的一半。  
- space-around：在每行上均匀分配弹性元素。相邻元素间距离相同。每行第一个元素与行首对齐，每行最后一个元素与行尾对齐。  
- stretch：均匀排列每个元素，'auto'-sized 的元素会被拉伸以适应容器的大小

#### align-items：在交叉轴方向上的对齐方式
取值：
- stretch：默认值，交叉轴方向拉伸填满 flex 容器
- center：居中
- flex-start: 元素向侧轴起点对齐
- flex-end: 元素向侧轴终点对齐