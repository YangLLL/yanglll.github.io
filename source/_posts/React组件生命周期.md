---
title: React组件生命周期
date: 2021-05-21 17:24:52
tags:
---
## class 组件的生命周期

### 挂载阶段
- constructor
  初始化 props 和 state

- componentWillMount
  在组件首次渲染之前调用，用的比较少，首次渲染之前的逻辑可以放 constructor 中处理

- render
  组件渲染更新

- componentDidMount
  组件首次渲染之后调用，只会调用一次，可以在这里发请求

### 更新阶段
- componentWillReceiveProps(nextProps)
  props 变化之后调用

- shouldComponentUpdate(nextProps, nextState)
  判断是否更新组件，返回 true 重新渲染，返回 false 不更新

- componentWillUpdate()
  组件更新之前调用

- render

- componentDidUpdate()
  组件更新之后调用

### 卸载阶段
- componentWillUnmount
  组件销毁之前调用，可以做一些清理工作，如清除定时器，事件监听等

### 新加的生命周期函数
- getDerivedStateFromProps(nextProps, prevState)
  会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 state，如果返回 null 则不更新任何内容。
  用来替换 componentWillReceiveProps，componentWillReceiveProps只会在父组件重新渲染的时候触发，这个方法每次渲染之前都会触发（包括 state 改变)。
  

- getSnapSotBeforeUpdate()
  在最近一次渲染输出（提交到 DOM 节点）之前调用。使组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期方法的任何返回值将作为参数传递给 componentDidUpdate()。
  在 render 之后，DOM 更新之前调用，替换 componentWillUpdate

### 过时的生命周期函数
- componentWillMount：组件挂载之前调用，基本用不到，初始化 state 的操作可以在 constructor 中处理
- componentWillReceiveProps：会在已挂载的组件接收新的 props 之前被调用
- componentWillUpdate：当组件收到新的 props 或 state 时，会在渲染之前调用，初始渲染的时候不会调用


## Hooks的生命周期
react hooks 的生命周期用 useEffect 模拟
```
useEffect(() => {
  // ...
}, dependency)
```
- useEffect() 第二个参数传 []，可以模拟 componentDidMount
- useEffect() 第二个参数传