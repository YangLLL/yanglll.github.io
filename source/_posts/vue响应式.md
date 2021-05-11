---
title: vue响应式
date: 2021-05-11 15:46:04
tags:
---

当数据发生变化时，会立刻触发视图的更新，就是响应式。

### Object.defineProperty()实现响应式
vue2.0是使用 Object.defineProperty() 来实现对 data 的监听。在渲染组件的时候，会收集依赖的数据，添加到 watcher 中。
对 data 的 value 进行 set 的时候，如果 data 里面的值发生了变化，就会通知 watcher，触发视图更新

##### 监听对象变化
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

深度监听：  
使用递归，如果是对象就继续调用defineProperty，重新定义对象

##### 监听数组变化
重新定义数组原型，创建新对象，原型指向原来的数组原型，在这基础上扩展数组的方法，然后将需要被监听的数组的原型赋值为我们自己定义的数组原型。
```
const arrayProperty = Array.prototype
const arrProto = Object.create(arrayProperty);
['push', 'pop', 'shift', 'unshift'].forEach(methodName => {
  arrProto[methodName] = function () {
    console.log('updata view------') // 触发视图更新
    arrayProperty[methodName].call(this, ...arguments)
  }
})
const arr = [1,2,3]
arr.__proto__ = arrProto
arr.push(4) // 打印 updata view------
```

##### Object.defineProperty() 的缺点：
- 深度监听，要一次性递归到底，对象里面的对象也要进行监听，如果对象比较大，一次性计算量会很大
- 不能监听增加，删除属性（设置新的响应式属性要使用 Vue.set 和 Vue.delete 来设置)
- 无法原生监听数组，需要特殊处理


### Proxy 实现响应式
vue3.0 使用了 Proxy 来实现响应式。

Proxy 的基本使用
```
const data = {
    name: 'zhangsan',
    age: 20,
}

const proxyData = new Proxy(data, {
  get(target, key, receiver) {
      const result = Reflect.get(target, key, receiver)
      console.log('get', key)
      return result // 返回结果
  },
  set(target, key, val, receiver) {
      const result = Reflect.set(target, key, val, receiver)
      console.log('set', key, val)
      return result // 是否设置成功
  },
  defineProperty(target, key) {
      const result = Reflect.deleteProperty(target, key)
      console.log('delete property', key)
      return result // 是否删除成功
  }
})
```

Proxy实现响应式的优点：
- 深度监听，性能更好：在 get 的时候递归，获取到哪一层，哪一层才变成响应式的，Object.defineProperty() 是在一开始就递归完了
- 可监听 新增、删除 属性
- 可监听数组变化

缺点：  
Proxy 无法兼容所有浏览器，无法 polyfill