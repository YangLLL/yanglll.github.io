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

使用 useEffect 可以在函数组件中执行副作用操作。useEffect 会在每次渲染之后调用（第一次渲染，和后面更新都会调用），相当于 componentDidMount 和 componentDidUpdate
```
useEffect(() => {
  // ...副作用操作
}, dependency)
```
默认函数组件没有生命周期，使用 useEffect 可以模拟生命周期。
- 如果只在挂载的时候执行一次，dependency 就传空数组 []，表示不依赖任何状态，props 和 state 变化不会重新执行useEffect
- 模拟 componentDidUpdate：
    1. dependency 不传任何参数，那每次更新都会重新执行 useEffect 中的代码
    2. dependency 传入依赖的 props 或者 state 属性，当前依赖变了之后，才会执行 useEffect 中的代码
- 如果要清除一些副作用，可以在 useEffect 中返回一个函数，React 会在执行清楚操作的时候调用这个函数。可以模拟 componentWillUnMount

```
useEffect(() => {
    console.log('开始监听') // state,props改变时会执行
    
     // 并不完全等同于 componentWillUnmount，props发生变化也会执行结束监听。
     // 返回的函数会在下一次 effect 执行之前被执行，不是只有被销毁的时候执行，update的时候也会执行
    return () => {
        console.log('结束监听') 
    }
})
```