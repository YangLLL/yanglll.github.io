---
title: vue响应式
date: 2021-05-11 15:46:04
tags:
---

vue 响应式就是数据发生变化，会立刻触发视图的更新。
vue2.0是使用 **Object.defineProperty()** 来实现对 data 的监听。在渲染组件的时候，会收集依赖的数据，添加到 watcher 中。
对 data 的 value 进行 set 的时候，如果 data 里面的值发生了变化，就会通知 watcher，触发视图更新
```
function defineObjProperty(data, key, value) {
  Object.defineProperty(data, key, {
    get() {
      return value
    },
    set(newValue) {
      if (newValue !== value) {
        value = newValue
        // 可以在这里更新视图
        console.log('update value------', newValue)
      }
    },
  })
}
```
