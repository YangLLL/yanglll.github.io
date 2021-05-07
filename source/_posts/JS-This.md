---
title: JS-This
date: 2021-05-07 22:11:49
tags:
---

## 1. this 是什么
this是当前执行上下文的一个属性，在全局环境中，this指向全局对象（window）。在函数内部，this的值取决于函数被调用的方式。 

默认情况下，this指向window。当函数作为一个对象的方法时，this指向调用函数的那个对象。如果是通过new调用的，this指向new生产的那个对象。通过bind、call、apply可以将this绑定到指定对象。

bind绑定this，不会马上调用，call和apply会马上调用。call接收一个参数列表，apply第二个参数是参数数组

箭头函数的this是在定义时绑定的，箭头函数的this和上层作用域的this值相同。（箭头函数不能作为构造函数）


## 2. this 绑定规则
#### 2.1 默认绑定

```
function foo() { 
    console.log( this.a );
}
var a = 2; 
foo(); // 2
```

函数直接调用，使用默认绑定。默认绑定的 this在非严格模式下指向全局对象。

#### 2.2 隐式绑定
当函数引 用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象。

```
function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
obj.foo(); // 2
```

foo的this被绑定到 obj 上了。

函数的 this 指向直接调用它的对象

```
function foo() { 
    console.log( this.a );
}
var obj2 = { 
    a: 42,
    foo: foo 
};
var obj1 = { 
    a: 2,
   obj2: obj2
};
obj1.obj2.foo(); // 42
```


隐式丢失：
被隐式绑定的函数丢失了绑定的对象，会应用默认绑定，把this绑定到全局对象 或 undefined 上。

```
function foo() { 
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo 
};
var bar = obj.foo; //函数别名!
var a = "oops, global"; // a 是全局对象的属性
bar(); // "oops, global"
```

bar 引用的是 foo 函数本身。在全局环境执行 bar()会使用默认绑定。

调用回调函数的函数可能会修改 this

```
function foo() { 
    console.log( this.a );
}
function doFoo(fn) {
    // fn 其实引用的是 foo, 默认绑定 window
    fn(); // <-- 调用位置!
}
var obj = {
     a: 2,
    foo: foo 
};
var a = "oops, global"; // a 是全局对象的属性
doFoo( obj.foo ); // "oops, global"

function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
var a = "oops, global"; // a 是全局对象的属性
setTimeout( obj.foo, 100 ); // "oops, global"
```

如果是传入 setTimeout 的回调 ，也是一样的

#### 2.3 显式绑定
使用call() 、apply() 和  bind()  使调用的函数的 this绑定到指定对象。

它们的第一个参数是一个对象，它们会把这个对象绑定到this，接着再调用函数时指定这个this

#### 2.4 new 绑定
构造函数：
在JavaScript中，构造函数只是一些使用new操作符时被调用的函数

使用new来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。
1. 创建(或者说构造)一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

```
function foo(a) {
    this.a = a;
}
var bar = new foo(2); 
console.log( bar.a ); // 2
```

使用 new 来调用 foo()  时，会构造一个新的对象，并把 foo() 的 this 指向这个新的对象。

## 3. 绑定优先级
 **new绑定 > 显式绑定 > 隐式绑定 > 默认绑定**

判断this：
- 函数在new中调用(new绑定)：this绑定的是新创建的对象。

- 函数通过call、apply(显式绑定)或者硬绑定(bind)调用：this绑定的是指定的对象。  

- 函数在某个上下文对象中调用(隐式绑定)this绑定的是那个上 下文对象。  

- 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。


## 4. 箭头函数
箭头函数不使用 this 的四种标准规则，而是根据外层(函数或者全局)作用域来决定 this。  
箭头函数不会创建自己的 this，它只会从自己的作用域链的上一层继承 this，是在定义的时候绑定的this  

```
function foo() {
    //返回一个箭头函数
    return(a) => {
        //this继承自foo()
        console.log(this.a ); 
    };
}
var obj1 = { 
    a:2
};
var obj2 = {
    a:3
};
var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是3!
```

foo() 内部创建的箭头函数会捕获调用时 foo() 的 this。由于 foo() 的 this 绑定到obj1，bar(引用箭头函数)的 this 也会绑定到 obj1，箭头函数的绑定无法被修改。(new 也不行!)

